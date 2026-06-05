# intrinsics.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Evaluate/intrinsics.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements compile-time Fortran expression evaluation logic and helpers for intrinsics.
- **Purpose (CN)**: 实现 intrinsics 相关的 Fortran 编译期表达式求值逻辑与辅助功能。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Evaluate/intrinsics.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Evaluate/intrinsics.h"
#include "flang/Common/enum-set.h"
#include "flang/Common/float128.h"
#include "flang/Common/idioms.h"
#include "flang/Evaluate/check-expression.h"
#include "flang/Evaluate/common.h"
#include "flang/Evaluate/expression.h"
#include "flang/Evaluate/fold.h"
#include "flang/Evaluate/shape.h"
#include "flang/Evaluate/tools.h"
#include "flang/Evaluate/type.h"
#include "flang/Semantics/scope.h"
#include "flang/Semantics/tools.h"
#include "flang/Support/Fortran.h"
#include "llvm/Support/raw_ostream.h"
#include <algorithm>
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, intent, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的逻辑、意图或元数据：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, intent, or metadata: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的逻辑、意图或元数据：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, intent, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的逻辑、意图或元数据：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes "flang/Evaluate/intrinsics.h" to access Fortran constant-folding and evaluation facilities.
  **L9 CN**: 引入 "flang/Evaluate/intrinsics.h" 以使用Fortran 常量折叠与求值能力。
- **L10 EN**: Includes "flang/Common/enum-set.h" to access shared Flang utility infrastructure.
  **L10 CN**: 引入 "flang/Common/enum-set.h" 以使用Flang 共享工具基础设施。
- **L11 EN**: Includes "flang/Common/float128.h" to access shared Flang utility infrastructure.
  **L11 CN**: 引入 "flang/Common/float128.h" 以使用Flang 共享工具基础设施。
- **L12 EN**: Includes "flang/Common/idioms.h" to access shared Flang utility infrastructure.
  **L12 CN**: 引入 "flang/Common/idioms.h" 以使用Flang 共享工具基础设施。
- **L13 EN**: Includes "flang/Evaluate/check-expression.h" to access Fortran constant-folding and evaluation facilities.
  **L13 CN**: 引入 "flang/Evaluate/check-expression.h" 以使用Fortran 常量折叠与求值能力。
- **L14 EN**: Includes "flang/Evaluate/common.h" to access Fortran constant-folding and evaluation facilities.
  **L14 CN**: 引入 "flang/Evaluate/common.h" 以使用Fortran 常量折叠与求值能力。
- **L15 EN**: Includes "flang/Evaluate/expression.h" to access Fortran constant-folding and evaluation facilities.
  **L15 CN**: 引入 "flang/Evaluate/expression.h" 以使用Fortran 常量折叠与求值能力。
- **L16 EN**: Includes "flang/Evaluate/fold.h" to access Fortran constant-folding and evaluation facilities.
  **L16 CN**: 引入 "flang/Evaluate/fold.h" 以使用Fortran 常量折叠与求值能力。
- **L17 EN**: Includes "flang/Evaluate/shape.h" to access Fortran constant-folding and evaluation facilities.
  **L17 CN**: 引入 "flang/Evaluate/shape.h" 以使用Fortran 常量折叠与求值能力。
- **L18 EN**: Includes "flang/Evaluate/tools.h" to access Fortran constant-folding and evaluation facilities.
  **L18 CN**: 引入 "flang/Evaluate/tools.h" 以使用Fortran 常量折叠与求值能力。
- **L19 EN**: Includes "flang/Evaluate/type.h" to access Fortran constant-folding and evaluation facilities.
  **L19 CN**: 引入 "flang/Evaluate/type.h" 以使用Fortran 常量折叠与求值能力。
- **L20 EN**: Includes "flang/Semantics/scope.h" to access Fortran semantic analysis, symbol, and type information.
  **L20 CN**: 引入 "flang/Semantics/scope.h" 以使用Fortran 语义分析、符号与类型信息。
- **L21 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L21 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L22 EN**: Includes "flang/Support/Fortran.h" to access shared Flang utility infrastructure.
  **L22 CN**: 引入 "flang/Support/Fortran.h" 以使用Flang 共享工具基础设施。
- **L23 EN**: Includes "llvm/Support/raw_ostream.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L23 CN**: 引入 "llvm/Support/raw_ostream.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L24 EN**: Includes <algorithm> to access supporting declarations used by this translation unit.
  **L24 CN**: 引入 <algorithm> 以使用当前编译单元使用的辅助声明。

### Lines 25-48

````cpp
#include <climits>
#include <cmath>
#include <map>
#include <string>
#include <utility>

using namespace Fortran::parser::literals;

namespace Fortran::evaluate {

class FoldingContext;

// This file defines the supported intrinsic procedures and implements
// their recognition and validation.  It is largely table-driven.  See
// docs/intrinsics.md and section 16 of the Fortran 2018 standard
// for full details on each of the intrinsics.  Be advised, they have
// complicated details, and the design of these tables has to accommodate
// that complexity.

// Dummy arguments to generic intrinsic procedures are each specified by
// their keyword name (rarely used, but always defined), allowable type
// categories, a kind pattern, a rank pattern, and information about
// optionality and defaults.  The kind and rank patterns are represented
// here with code values that are significant to the matching/validation engine.
````
- **L25 EN**: Includes <climits> to access supporting declarations used by this translation unit.
  **L25 CN**: 引入 <climits> 以使用当前编译单元使用的辅助声明。
- **L26 EN**: Includes <cmath> to access supporting declarations used by this translation unit.
  **L26 CN**: 引入 <cmath> 以使用当前编译单元使用的辅助声明。
- **L27 EN**: Includes <map> to access supporting declarations used by this translation unit.
  **L27 CN**: 引入 <map> 以使用当前编译单元使用的辅助声明。
- **L28 EN**: Includes <string> to access supporting declarations used by this translation unit.
  **L28 CN**: 引入 <string> 以使用当前编译单元使用的辅助声明。
- **L29 EN**: Includes <utility> to access supporting declarations used by this translation unit.
  **L29 CN**: 引入 <utility> 以使用当前编译单元使用的辅助声明。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L31 EN**: Brings namespace `Fortran::parser::literals` into the local scope.
  **L31 CN**: 将命名空间 `Fortran::parser::literals` 引入当前作用域。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Opens namespace scope `Fortran::evaluate`.
  **L33 CN**: 打开命名空间作用域 `Fortran::evaluate`。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Declares class `FoldingContext;`.
  **L35 CN**: 声明 class `FoldingContext;`。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L37 EN**: Comment explains nearby logic, intent, or metadata: `This file defines the supported intrinsic procedures and implements`.
  **L37 CN**: 注释说明附近代码的逻辑、意图或元数据：`This file defines the supported intrinsic procedures and implements`。
- **L38 EN**: Comment explains nearby logic, intent, or metadata: `their recognition and validation.  It is largely table-driven.  See`.
  **L38 CN**: 注释说明附近代码的逻辑、意图或元数据：`their recognition and validation.  It is largely table-driven.  See`。
- **L39 EN**: Comment explains nearby logic, intent, or metadata: `docs/intrinsics.md and section 16 of the Fortran 2018 standard`.
  **L39 CN**: 注释说明附近代码的逻辑、意图或元数据：`docs/intrinsics.md and section 16 of the Fortran 2018 standard`。
- **L40 EN**: Comment explains nearby logic, intent, or metadata: `for full details on each of the intrinsics.  Be advised, they have`.
  **L40 CN**: 注释说明附近代码的逻辑、意图或元数据：`for full details on each of the intrinsics.  Be advised, they have`。
- **L41 EN**: Comment explains nearby logic, intent, or metadata: `complicated details, and the design of these tables has to accommodate`.
  **L41 CN**: 注释说明附近代码的逻辑、意图或元数据：`complicated details, and the design of these tables has to accommodate`。
- **L42 EN**: Comment explains nearby logic, intent, or metadata: `that complexity.`.
  **L42 CN**: 注释说明附近代码的逻辑、意图或元数据：`that complexity.`。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Comment explains nearby logic, intent, or metadata: `Dummy arguments to generic intrinsic procedures are each specified by`.
  **L44 CN**: 注释说明附近代码的逻辑、意图或元数据：`Dummy arguments to generic intrinsic procedures are each specified by`。
- **L45 EN**: Comment explains nearby logic, intent, or metadata: `their keyword name (rarely used, but always defined), allowable type`.
  **L45 CN**: 注释说明附近代码的逻辑、意图或元数据：`their keyword name (rarely used, but always defined), allowable type`。
- **L46 EN**: Comment explains nearby logic, intent, or metadata: `categories, a kind pattern, a rank pattern, and information about`.
  **L46 CN**: 注释说明附近代码的逻辑、意图或元数据：`categories, a kind pattern, a rank pattern, and information about`。
- **L47 EN**: Comment explains nearby logic, intent, or metadata: `optionality and defaults.  The kind and rank patterns are represented`.
  **L47 CN**: 注释说明附近代码的逻辑、意图或元数据：`optionality and defaults.  The kind and rank patterns are represented`。
- **L48 EN**: Comment explains nearby logic, intent, or metadata: `here with code values that are significant to the matching/validation engine.`.
  **L48 CN**: 注释说明附近代码的逻辑、意图或元数据：`here with code values that are significant to the matching/validation engine.`。

### Lines 49-72

````cpp

// An actual argument to an intrinsic procedure may be a procedure itself
// only if the dummy argument is Rank::reduceOperation,
// KindCode::addressable, or the special case of NULL(MOLD=procedurePointer).

// These are small bit-sets of type category enumerators.
// Note that typeless (BOZ literal) values don't have a distinct type category.
// These typeless arguments are represented in the tables as if they were
// INTEGER with a special "typeless" kind code.  Arguments of intrinsic types
// that can also be typeless values are encoded with an "elementalOrBOZ"
// rank pattern.
// Assumed-type (TYPE(*)) dummy arguments can be forwarded along to some
// intrinsic functions that accept AnyType + Rank::anyOrAssumedRank,
// AnyType + Rank::arrayOrAssumedRank,  or AnyType + Kind::addressable.
using CategorySet = common::EnumSet<TypeCategory, 8>;
static constexpr CategorySet IntType{TypeCategory::Integer};
static constexpr CategorySet UnsignedType{TypeCategory::Unsigned};
static constexpr CategorySet RealType{TypeCategory::Real};
static constexpr CategorySet ComplexType{TypeCategory::Complex};
static constexpr CategorySet CharType{TypeCategory::Character};
static constexpr CategorySet LogicalType{TypeCategory::Logical};
static constexpr CategorySet IntOrUnsignedType{IntType | UnsignedType};
static constexpr CategorySet IntOrRealType{IntType | RealType};
static constexpr CategorySet IntUnsignedOrRealType{
````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Comment explains nearby logic, intent, or metadata: `An actual argument to an intrinsic procedure may be a procedure itself`.
  **L50 CN**: 注释说明附近代码的逻辑、意图或元数据：`An actual argument to an intrinsic procedure may be a procedure itself`。
- **L51 EN**: Comment explains nearby logic, intent, or metadata: `only if the dummy argument is Rank::reduceOperation,`.
  **L51 CN**: 注释说明附近代码的逻辑、意图或元数据：`only if the dummy argument is Rank::reduceOperation,`。
- **L52 EN**: Comment explains nearby logic, intent, or metadata: `KindCode::addressable, or the special case of NULL(MOLD=procedurePointer).`.
  **L52 CN**: 注释说明附近代码的逻辑、意图或元数据：`KindCode::addressable, or the special case of NULL(MOLD=procedurePointer).`。
- **L53 EN**: Blank line separating nearby declarations or logic blocks.
  **L53 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L54 EN**: Comment explains nearby logic, intent, or metadata: `These are small bit-sets of type category enumerators.`.
  **L54 CN**: 注释说明附近代码的逻辑、意图或元数据：`These are small bit-sets of type category enumerators.`。
- **L55 EN**: Comment explains nearby logic, intent, or metadata: `Note that typeless (BOZ literal) values don't have a distinct type category.`.
  **L55 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that typeless (BOZ literal) values don't have a distinct type category.`。
- **L56 EN**: Comment explains nearby logic, intent, or metadata: `These typeless arguments are represented in the tables as if they were`.
  **L56 CN**: 注释说明附近代码的逻辑、意图或元数据：`These typeless arguments are represented in the tables as if they were`。
- **L57 EN**: Comment explains nearby logic, intent, or metadata: `INTEGER with a special "typeless" kind code.  Arguments of intrinsic types`.
  **L57 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTEGER with a special "typeless" kind code.  Arguments of intrinsic types`。
- **L58 EN**: Comment explains nearby logic, intent, or metadata: `that can also be typeless values are encoded with an "elementalOrBOZ"`.
  **L58 CN**: 注释说明附近代码的逻辑、意图或元数据：`that can also be typeless values are encoded with an "elementalOrBOZ"`。
- **L59 EN**: Comment explains nearby logic, intent, or metadata: `rank pattern.`.
  **L59 CN**: 注释说明附近代码的逻辑、意图或元数据：`rank pattern.`。
- **L60 EN**: Comment explains nearby logic, intent, or metadata: `Assumed-type (TYPE(*)) dummy arguments can be forwarded along to some`.
  **L60 CN**: 注释说明附近代码的逻辑、意图或元数据：`Assumed-type (TYPE(*)) dummy arguments can be forwarded along to some`。
- **L61 EN**: Comment explains nearby logic, intent, or metadata: `intrinsic functions that accept AnyType + Rank::anyOrAssumedRank,`.
  **L61 CN**: 注释说明附近代码的逻辑、意图或元数据：`intrinsic functions that accept AnyType + Rank::anyOrAssumedRank,`。
- **L62 EN**: Comment explains nearby logic, intent, or metadata: `AnyType + Rank::arrayOrAssumedRank,  or AnyType + Kind::addressable.`.
  **L62 CN**: 注释说明附近代码的逻辑、意图或元数据：`AnyType + Rank::arrayOrAssumedRank,  or AnyType + Kind::addressable.`。
- **L63 EN**: Defines alias `CategorySet` to simplify later code.
  **L63 CN**: 定义别名 `CategorySet` 以简化后续代码。
- **L64 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet IntType{TypeCategory::Integer};`.
  **L64 CN**: 执行一条独立语句或声明：`static constexpr CategorySet IntType{TypeCategory::Integer};`。
- **L65 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet UnsignedType{TypeCategory::Unsigned};`.
  **L65 CN**: 执行一条独立语句或声明：`static constexpr CategorySet UnsignedType{TypeCategory::Unsigned};`。
- **L66 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet RealType{TypeCategory::Real};`.
  **L66 CN**: 执行一条独立语句或声明：`static constexpr CategorySet RealType{TypeCategory::Real};`。
- **L67 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet ComplexType{TypeCategory::Complex};`.
  **L67 CN**: 执行一条独立语句或声明：`static constexpr CategorySet ComplexType{TypeCategory::Complex};`。
- **L68 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet CharType{TypeCategory::Character};`.
  **L68 CN**: 执行一条独立语句或声明：`static constexpr CategorySet CharType{TypeCategory::Character};`。
- **L69 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet LogicalType{TypeCategory::Logical};`.
  **L69 CN**: 执行一条独立语句或声明：`static constexpr CategorySet LogicalType{TypeCategory::Logical};`。
- **L70 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet IntOrUnsignedType{IntType | UnsignedType};`.
  **L70 CN**: 执行一条独立语句或声明：`static constexpr CategorySet IntOrUnsignedType{IntType | UnsignedType};`。
- **L71 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet IntOrRealType{IntType | RealType};`.
  **L71 CN**: 执行一条独立语句或声明：`static constexpr CategorySet IntOrRealType{IntType | RealType};`。
- **L72 EN**: Continues the surrounding expression or declaration: `static constexpr CategorySet IntUnsignedOrRealType{`.
  **L72 CN**: 继续构造周围的表达式或声明：`static constexpr CategorySet IntUnsignedOrRealType{`。

### Lines 73-96

````cpp
    IntType | UnsignedType | RealType};
static constexpr CategorySet IntOrRealOrCharType{IntType | RealType | CharType};
static constexpr CategorySet IntOrLogicalType{IntType | LogicalType};
static constexpr CategorySet FloatingType{RealType | ComplexType};
static constexpr CategorySet NumericType{
    IntType | UnsignedType | RealType | ComplexType};
static constexpr CategorySet RelatableType{
    IntType | UnsignedType | RealType | CharType};
static constexpr CategorySet DerivedType{TypeCategory::Derived};
static constexpr CategorySet IntrinsicType{
    IntType | UnsignedType | RealType | ComplexType | CharType | LogicalType};
static constexpr CategorySet AnyType{IntrinsicType | DerivedType};

ENUM_CLASS(KindCode, none, defaultIntegerKind,
    defaultRealKind, // is also the default COMPLEX kind
    doublePrecision, quadPrecision, defaultCharKind, defaultLogicalKind,
    greaterOrEqualToKind, // match kind value greater than or equal to a single
                          // explicit kind value
    any, // matches any kind value; each instance is independent
    // match any kind, but all "same" kinds must be equal. For characters, also
    // implies that lengths must be equal.
    same,
    // for characters that only require the same kind, not length
    sameKind,
````
- **L73 EN**: Executes a standalone statement or declaration: `IntType | UnsignedType | RealType};`.
  **L73 CN**: 执行一条独立语句或声明：`IntType | UnsignedType | RealType};`。
- **L74 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet IntOrRealOrCharType{IntType | RealType | CharType};`.
  **L74 CN**: 执行一条独立语句或声明：`static constexpr CategorySet IntOrRealOrCharType{IntType | RealType | CharType};`。
- **L75 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet IntOrLogicalType{IntType | LogicalType};`.
  **L75 CN**: 执行一条独立语句或声明：`static constexpr CategorySet IntOrLogicalType{IntType | LogicalType};`。
- **L76 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet FloatingType{RealType | ComplexType};`.
  **L76 CN**: 执行一条独立语句或声明：`static constexpr CategorySet FloatingType{RealType | ComplexType};`。
- **L77 EN**: Continues the surrounding expression or declaration: `static constexpr CategorySet NumericType{`.
  **L77 CN**: 继续构造周围的表达式或声明：`static constexpr CategorySet NumericType{`。
- **L78 EN**: Executes a standalone statement or declaration: `IntType | UnsignedType | RealType | ComplexType};`.
  **L78 CN**: 执行一条独立语句或声明：`IntType | UnsignedType | RealType | ComplexType};`。
- **L79 EN**: Continues the surrounding expression or declaration: `static constexpr CategorySet RelatableType{`.
  **L79 CN**: 继续构造周围的表达式或声明：`static constexpr CategorySet RelatableType{`。
- **L80 EN**: Executes a standalone statement or declaration: `IntType | UnsignedType | RealType | CharType};`.
  **L80 CN**: 执行一条独立语句或声明：`IntType | UnsignedType | RealType | CharType};`。
- **L81 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet DerivedType{TypeCategory::Derived};`.
  **L81 CN**: 执行一条独立语句或声明：`static constexpr CategorySet DerivedType{TypeCategory::Derived};`。
- **L82 EN**: Continues the surrounding expression or declaration: `static constexpr CategorySet IntrinsicType{`.
  **L82 CN**: 继续构造周围的表达式或声明：`static constexpr CategorySet IntrinsicType{`。
- **L83 EN**: Executes a standalone statement or declaration: `IntType | UnsignedType | RealType | ComplexType | CharType | LogicalType};`.
  **L83 CN**: 执行一条独立语句或声明：`IntType | UnsignedType | RealType | ComplexType | CharType | LogicalType};`。
- **L84 EN**: Executes a standalone statement or declaration: `static constexpr CategorySet AnyType{IntrinsicType | DerivedType};`.
  **L84 CN**: 执行一条独立语句或声明：`static constexpr CategorySet AnyType{IntrinsicType | DerivedType};`。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L86 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ENUM_CLASS(KindCode, none, defaultIntegerKind,`.
  **L86 CN**: 继续一个多行参数列表、初始化器或聚合项：`ENUM_CLASS(KindCode, none, defaultIntegerKind,`。
- **L87 EN**: Continues the surrounding expression or declaration: `defaultRealKind, // is also the default COMPLEX kind`.
  **L87 CN**: 继续构造周围的表达式或声明：`defaultRealKind, // is also the default COMPLEX kind`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `doublePrecision, quadPrecision, defaultCharKind, defaultLogicalKind,`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`doublePrecision, quadPrecision, defaultCharKind, defaultLogicalKind,`。
- **L89 EN**: Continues the surrounding expression or declaration: `greaterOrEqualToKind, // match kind value greater than or equal to a single`.
  **L89 CN**: 继续构造周围的表达式或声明：`greaterOrEqualToKind, // match kind value greater than or equal to a single`。
- **L90 EN**: Comment explains nearby logic, intent, or metadata: `explicit kind value`.
  **L90 CN**: 注释说明附近代码的逻辑、意图或元数据：`explicit kind value`。
- **L91 EN**: Continues the surrounding expression or declaration: `any, // matches any kind value; each instance is independent`.
  **L91 CN**: 继续构造周围的表达式或声明：`any, // matches any kind value; each instance is independent`。
- **L92 EN**: Comment explains nearby logic, intent, or metadata: `match any kind, but all "same" kinds must be equal. For characters, also`.
  **L92 CN**: 注释说明附近代码的逻辑、意图或元数据：`match any kind, but all "same" kinds must be equal. For characters, also`。
- **L93 EN**: Comment explains nearby logic, intent, or metadata: `implies that lengths must be equal.`.
  **L93 CN**: 注释说明附近代码的逻辑、意图或元数据：`implies that lengths must be equal.`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `same,`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`same,`。
- **L95 EN**: Comment explains nearby logic, intent, or metadata: `for characters that only require the same kind, not length`.
  **L95 CN**: 注释说明附近代码的逻辑、意图或元数据：`for characters that only require the same kind, not length`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `sameKind,`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`sameKind,`。

### Lines 97-120

````cpp
    operand, // match any kind, with promotion (non-standard)
    typeless, // BOZ literals are INTEGER with this kind
    ieeeFlagType, // IEEE_FLAG_TYPE from ISO_FORTRAN_EXCEPTION
    ieeeRoundType, // IEEE_ROUND_TYPE from ISO_FORTRAN_ARITHMETIC
    eventType, // EVENT_TYPE from module ISO_FORTRAN_ENV (for coarrays)
    teamType, // TEAM_TYPE from module ISO_FORTRAN_ENV (for coarrays)
    kindArg, // this argument is KIND=
    effectiveKind, // for function results: "kindArg" value, possibly defaulted
    dimArg, // this argument is DIM=
    likeMultiply, // for DOT_PRODUCT and MATMUL
    subscript, // address-sized integer
    size, // default KIND= for SIZE(), UBOUND, &c.
    addressable, // for PRESENT(), &c.; anything (incl. procedure) but BOZ
    nullPointerType, // for ASSOCIATED(NULL())
    exactKind, // a single explicit exactKindValue
    atomicIntKind, // atomic_int_kind from iso_fortran_env
    atomicIntOrLogicalKind, // atomic_int_kind or atomic_logical_kind
    sameAtom, // same type and kind as atom
    extensibleOrUnlimitedType, // extensible or unlimited polymorphic type
)

struct TypePattern {
  CategorySet categorySet;
  KindCode kindCode{KindCode::none};
````
- **L97 EN**: Continues logic associated with callable symbol `promotion`.
  **L97 CN**: 继续与可调用符号 `promotion` 相关的逻辑。
- **L98 EN**: Continues the surrounding expression or declaration: `typeless, // BOZ literals are INTEGER with this kind`.
  **L98 CN**: 继续构造周围的表达式或声明：`typeless, // BOZ literals are INTEGER with this kind`。
- **L99 EN**: Continues the surrounding expression or declaration: `ieeeFlagType, // IEEE_FLAG_TYPE from ISO_FORTRAN_EXCEPTION`.
  **L99 CN**: 继续构造周围的表达式或声明：`ieeeFlagType, // IEEE_FLAG_TYPE from ISO_FORTRAN_EXCEPTION`。
- **L100 EN**: Continues the surrounding expression or declaration: `ieeeRoundType, // IEEE_ROUND_TYPE from ISO_FORTRAN_ARITHMETIC`.
  **L100 CN**: 继续构造周围的表达式或声明：`ieeeRoundType, // IEEE_ROUND_TYPE from ISO_FORTRAN_ARITHMETIC`。
- **L101 EN**: Continues logic associated with callable symbol `ISO_FORTRAN_ENV`.
  **L101 CN**: 继续与可调用符号 `ISO_FORTRAN_ENV` 相关的逻辑。
- **L102 EN**: Continues logic associated with callable symbol `ISO_FORTRAN_ENV`.
  **L102 CN**: 继续与可调用符号 `ISO_FORTRAN_ENV` 相关的逻辑。
- **L103 EN**: Continues the surrounding expression or declaration: `kindArg, // this argument is KIND=`.
  **L103 CN**: 继续构造周围的表达式或声明：`kindArg, // this argument is KIND=`。
- **L104 EN**: Continues the surrounding expression or declaration: `effectiveKind, // for function results: "kindArg" value, possibly defaulted`.
  **L104 CN**: 继续构造周围的表达式或声明：`effectiveKind, // for function results: "kindArg" value, possibly defaulted`。
- **L105 EN**: Continues the surrounding expression or declaration: `dimArg, // this argument is DIM=`.
  **L105 CN**: 继续构造周围的表达式或声明：`dimArg, // this argument is DIM=`。
- **L106 EN**: Continues the surrounding expression or declaration: `likeMultiply, // for DOT_PRODUCT and MATMUL`.
  **L106 CN**: 继续构造周围的表达式或声明：`likeMultiply, // for DOT_PRODUCT and MATMUL`。
- **L107 EN**: Continues the surrounding expression or declaration: `subscript, // address-sized integer`.
  **L107 CN**: 继续构造周围的表达式或声明：`subscript, // address-sized integer`。
- **L108 EN**: Continues logic associated with callable symbol `SIZE`.
  **L108 CN**: 继续与可调用符号 `SIZE` 相关的逻辑。
- **L109 EN**: Continues logic associated with callable symbol `PRESENT`.
  **L109 CN**: 继续与可调用符号 `PRESENT` 相关的逻辑。
- **L110 EN**: Continues logic associated with callable symbol `ASSOCIATED`.
  **L110 CN**: 继续与可调用符号 `ASSOCIATED` 相关的逻辑。
- **L111 EN**: Continues the surrounding expression or declaration: `exactKind, // a single explicit exactKindValue`.
  **L111 CN**: 继续构造周围的表达式或声明：`exactKind, // a single explicit exactKindValue`。
- **L112 EN**: Continues the surrounding expression or declaration: `atomicIntKind, // atomic_int_kind from iso_fortran_env`.
  **L112 CN**: 继续构造周围的表达式或声明：`atomicIntKind, // atomic_int_kind from iso_fortran_env`。
- **L113 EN**: Continues the surrounding expression or declaration: `atomicIntOrLogicalKind, // atomic_int_kind or atomic_logical_kind`.
  **L113 CN**: 继续构造周围的表达式或声明：`atomicIntOrLogicalKind, // atomic_int_kind or atomic_logical_kind`。
- **L114 EN**: Continues the surrounding expression or declaration: `sameAtom, // same type and kind as atom`.
  **L114 CN**: 继续构造周围的表达式或声明：`sameAtom, // same type and kind as atom`。
- **L115 EN**: Continues the surrounding expression or declaration: `extensibleOrUnlimitedType, // extensible or unlimited polymorphic type`.
  **L115 CN**: 继续构造周围的表达式或声明：`extensibleOrUnlimitedType, // extensible or unlimited polymorphic type`。
- **L116 EN**: Continues the surrounding expression or declaration: `)`.
  **L116 CN**: 继续构造周围的表达式或声明：`)`。
- **L117 EN**: Blank line separating nearby declarations or logic blocks.
  **L117 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L118 EN**: Declares struct `TypePattern`.
  **L118 CN**: 声明 struct `TypePattern`。
- **L119 EN**: Executes a standalone statement or declaration: `CategorySet categorySet;`.
  **L119 CN**: 执行一条独立语句或声明：`CategorySet categorySet;`。
- **L120 EN**: Executes a standalone statement or declaration: `KindCode kindCode{KindCode::none};`.
  **L120 CN**: 执行一条独立语句或声明：`KindCode kindCode{KindCode::none};`。

### Lines 121-144

````cpp
  int kindValue{0}; // for KindCode::exactKind and greaterOrEqualToKind
  llvm::raw_ostream &Dump(llvm::raw_ostream &) const;
};

// Abbreviations for argument and result patterns in the intrinsic prototypes:

// Match specific kinds of intrinsic types
static constexpr TypePattern DefaultInt{IntType, KindCode::defaultIntegerKind};
static constexpr TypePattern DefaultReal{RealType, KindCode::defaultRealKind};
static constexpr TypePattern DefaultComplex{
    ComplexType, KindCode::defaultRealKind};
static constexpr TypePattern DefaultChar{CharType, KindCode::defaultCharKind};
static constexpr TypePattern DefaultLogical{
    LogicalType, KindCode::defaultLogicalKind};
static constexpr TypePattern BOZ{IntType, KindCode::typeless};
static constexpr TypePattern CChar{CharType, KindCode::defaultCharKind};
static constexpr TypePattern EventType{DerivedType, KindCode::eventType};
static constexpr TypePattern IeeeFlagType{DerivedType, KindCode::ieeeFlagType};
static constexpr TypePattern IeeeRoundType{
    DerivedType, KindCode::ieeeRoundType};
static constexpr TypePattern TeamType{DerivedType, KindCode::teamType};
static constexpr TypePattern DoublePrecision{
    RealType, KindCode::doublePrecision};
static constexpr TypePattern DoublePrecisionComplex{
````
- **L121 EN**: Continues the surrounding expression or declaration: `int kindValue{0}; // for KindCode::exactKind and greaterOrEqualToKind`.
  **L121 CN**: 继续构造周围的表达式或声明：`int kindValue{0}; // for KindCode::exactKind and greaterOrEqualToKind`。
- **L122 EN**: Executes a call or declaration centered on `&Dump`.
  **L122 CN**: 执行以 `&Dump` 为核心的调用或声明。
- **L123 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L123 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L125 EN**: Comment explains nearby logic, intent, or metadata: `Abbreviations for argument and result patterns in the intrinsic prototypes:`.
  **L125 CN**: 注释说明附近代码的逻辑、意图或元数据：`Abbreviations for argument and result patterns in the intrinsic prototypes:`。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L127 EN**: Comment explains nearby logic, intent, or metadata: `Match specific kinds of intrinsic types`.
  **L127 CN**: 注释说明附近代码的逻辑、意图或元数据：`Match specific kinds of intrinsic types`。
- **L128 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern DefaultInt{IntType, KindCode::defaultIntegerKind};`.
  **L128 CN**: 执行一条独立语句或声明：`static constexpr TypePattern DefaultInt{IntType, KindCode::defaultIntegerKind};`。
- **L129 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern DefaultReal{RealType, KindCode::defaultRealKind};`.
  **L129 CN**: 执行一条独立语句或声明：`static constexpr TypePattern DefaultReal{RealType, KindCode::defaultRealKind};`。
- **L130 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern DefaultComplex{`.
  **L130 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern DefaultComplex{`。
- **L131 EN**: Executes a standalone statement or declaration: `ComplexType, KindCode::defaultRealKind};`.
  **L131 CN**: 执行一条独立语句或声明：`ComplexType, KindCode::defaultRealKind};`。
- **L132 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern DefaultChar{CharType, KindCode::defaultCharKind};`.
  **L132 CN**: 执行一条独立语句或声明：`static constexpr TypePattern DefaultChar{CharType, KindCode::defaultCharKind};`。
- **L133 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern DefaultLogical{`.
  **L133 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern DefaultLogical{`。
- **L134 EN**: Executes a standalone statement or declaration: `LogicalType, KindCode::defaultLogicalKind};`.
  **L134 CN**: 执行一条独立语句或声明：`LogicalType, KindCode::defaultLogicalKind};`。
- **L135 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern BOZ{IntType, KindCode::typeless};`.
  **L135 CN**: 执行一条独立语句或声明：`static constexpr TypePattern BOZ{IntType, KindCode::typeless};`。
- **L136 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern CChar{CharType, KindCode::defaultCharKind};`.
  **L136 CN**: 执行一条独立语句或声明：`static constexpr TypePattern CChar{CharType, KindCode::defaultCharKind};`。
- **L137 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern EventType{DerivedType, KindCode::eventType};`.
  **L137 CN**: 执行一条独立语句或声明：`static constexpr TypePattern EventType{DerivedType, KindCode::eventType};`。
- **L138 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern IeeeFlagType{DerivedType, KindCode::ieeeFlagType};`.
  **L138 CN**: 执行一条独立语句或声明：`static constexpr TypePattern IeeeFlagType{DerivedType, KindCode::ieeeFlagType};`。
- **L139 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern IeeeRoundType{`.
  **L139 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern IeeeRoundType{`。
- **L140 EN**: Executes a standalone statement or declaration: `DerivedType, KindCode::ieeeRoundType};`.
  **L140 CN**: 执行一条独立语句或声明：`DerivedType, KindCode::ieeeRoundType};`。
- **L141 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern TeamType{DerivedType, KindCode::teamType};`.
  **L141 CN**: 执行一条独立语句或声明：`static constexpr TypePattern TeamType{DerivedType, KindCode::teamType};`。
- **L142 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern DoublePrecision{`.
  **L142 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern DoublePrecision{`。
- **L143 EN**: Executes a standalone statement or declaration: `RealType, KindCode::doublePrecision};`.
  **L143 CN**: 执行一条独立语句或声明：`RealType, KindCode::doublePrecision};`。
- **L144 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern DoublePrecisionComplex{`.
  **L144 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern DoublePrecisionComplex{`。

### Lines 145-168

````cpp
    ComplexType, KindCode::doublePrecision};
static constexpr TypePattern QuadPrecision{RealType, KindCode::quadPrecision};
static constexpr TypePattern SubscriptInt{IntType, KindCode::subscript};

// Match any kind of some intrinsic or derived types
static constexpr TypePattern AnyInt{IntType, KindCode::any};
static constexpr TypePattern AnyIntOrUnsigned{IntOrUnsignedType, KindCode::any};
static constexpr TypePattern AnyReal{RealType, KindCode::any};
static constexpr TypePattern AnyIntOrReal{IntOrRealType, KindCode::any};
static constexpr TypePattern AnyIntUnsignedOrReal{
    IntUnsignedOrRealType, KindCode::any};
static constexpr TypePattern AnyIntOrRealOrChar{
    IntOrRealOrCharType, KindCode::any};
static constexpr TypePattern AnyIntOrLogical{IntOrLogicalType, KindCode::any};
static constexpr TypePattern AnyComplex{ComplexType, KindCode::any};
static constexpr TypePattern AnyFloating{FloatingType, KindCode::any};
static constexpr TypePattern AnyNumeric{NumericType, KindCode::any};
static constexpr TypePattern AnyChar{CharType, KindCode::any};
static constexpr TypePattern AnyLogical{LogicalType, KindCode::any};
static constexpr TypePattern AnyRelatable{RelatableType, KindCode::any};
static constexpr TypePattern AnyIntrinsic{IntrinsicType, KindCode::any};
static constexpr TypePattern ExtensibleDerived{
    DerivedType, KindCode::extensibleOrUnlimitedType};
static constexpr TypePattern AnyData{AnyType, KindCode::any};
````
- **L145 EN**: Executes a standalone statement or declaration: `ComplexType, KindCode::doublePrecision};`.
  **L145 CN**: 执行一条独立语句或声明：`ComplexType, KindCode::doublePrecision};`。
- **L146 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern QuadPrecision{RealType, KindCode::quadPrecision};`.
  **L146 CN**: 执行一条独立语句或声明：`static constexpr TypePattern QuadPrecision{RealType, KindCode::quadPrecision};`。
- **L147 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SubscriptInt{IntType, KindCode::subscript};`.
  **L147 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SubscriptInt{IntType, KindCode::subscript};`。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L149 EN**: Comment explains nearby logic, intent, or metadata: `Match any kind of some intrinsic or derived types`.
  **L149 CN**: 注释说明附近代码的逻辑、意图或元数据：`Match any kind of some intrinsic or derived types`。
- **L150 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyInt{IntType, KindCode::any};`.
  **L150 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyInt{IntType, KindCode::any};`。
- **L151 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyIntOrUnsigned{IntOrUnsignedType, KindCode::any};`.
  **L151 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyIntOrUnsigned{IntOrUnsignedType, KindCode::any};`。
- **L152 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyReal{RealType, KindCode::any};`.
  **L152 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyReal{RealType, KindCode::any};`。
- **L153 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyIntOrReal{IntOrRealType, KindCode::any};`.
  **L153 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyIntOrReal{IntOrRealType, KindCode::any};`。
- **L154 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern AnyIntUnsignedOrReal{`.
  **L154 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern AnyIntUnsignedOrReal{`。
- **L155 EN**: Executes a standalone statement or declaration: `IntUnsignedOrRealType, KindCode::any};`.
  **L155 CN**: 执行一条独立语句或声明：`IntUnsignedOrRealType, KindCode::any};`。
- **L156 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern AnyIntOrRealOrChar{`.
  **L156 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern AnyIntOrRealOrChar{`。
- **L157 EN**: Executes a standalone statement or declaration: `IntOrRealOrCharType, KindCode::any};`.
  **L157 CN**: 执行一条独立语句或声明：`IntOrRealOrCharType, KindCode::any};`。
- **L158 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyIntOrLogical{IntOrLogicalType, KindCode::any};`.
  **L158 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyIntOrLogical{IntOrLogicalType, KindCode::any};`。
- **L159 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyComplex{ComplexType, KindCode::any};`.
  **L159 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyComplex{ComplexType, KindCode::any};`。
- **L160 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyFloating{FloatingType, KindCode::any};`.
  **L160 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyFloating{FloatingType, KindCode::any};`。
- **L161 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyNumeric{NumericType, KindCode::any};`.
  **L161 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyNumeric{NumericType, KindCode::any};`。
- **L162 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyChar{CharType, KindCode::any};`.
  **L162 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyChar{CharType, KindCode::any};`。
- **L163 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyLogical{LogicalType, KindCode::any};`.
  **L163 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyLogical{LogicalType, KindCode::any};`。
- **L164 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyRelatable{RelatableType, KindCode::any};`.
  **L164 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyRelatable{RelatableType, KindCode::any};`。
- **L165 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyIntrinsic{IntrinsicType, KindCode::any};`.
  **L165 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyIntrinsic{IntrinsicType, KindCode::any};`。
- **L166 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern ExtensibleDerived{`.
  **L166 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern ExtensibleDerived{`。
- **L167 EN**: Executes a standalone statement or declaration: `DerivedType, KindCode::extensibleOrUnlimitedType};`.
  **L167 CN**: 执行一条独立语句或声明：`DerivedType, KindCode::extensibleOrUnlimitedType};`。
- **L168 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyData{AnyType, KindCode::any};`.
  **L168 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyData{AnyType, KindCode::any};`。

### Lines 169-192

````cpp

// Type is irrelevant, but not BOZ (for PRESENT(), OPTIONAL(), &c.)
static constexpr TypePattern Addressable{AnyType, KindCode::addressable};

// Match some kind of some intrinsic type(s); all "Same" values must match,
// even when not in the same category (e.g., SameComplex and SameReal).
// Can be used to specify a result so long as at least one argument is
// a "Same".
static constexpr TypePattern SameInt{IntType, KindCode::same};
static constexpr TypePattern SameIntOrUnsigned{
    IntOrUnsignedType, KindCode::same};
static constexpr TypePattern SameReal{RealType, KindCode::same};
static constexpr TypePattern SameIntOrReal{IntOrRealType, KindCode::same};
static constexpr TypePattern SameIntUnsignedOrReal{
    IntUnsignedOrRealType, KindCode::same};
static constexpr TypePattern SameComplex{ComplexType, KindCode::same};
static constexpr TypePattern SameFloating{FloatingType, KindCode::same};
static constexpr TypePattern SameNumeric{NumericType, KindCode::same};
static constexpr TypePattern SameChar{CharType, KindCode::same};
static constexpr TypePattern SameCharNoLen{CharType, KindCode::sameKind};
static constexpr TypePattern SameLogical{LogicalType, KindCode::same};
static constexpr TypePattern SameRelatable{RelatableType, KindCode::same};
static constexpr TypePattern SameIntrinsic{IntrinsicType, KindCode::same};
static constexpr TypePattern SameType{AnyType, KindCode::same};
````
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L170 EN**: Comment explains nearby logic, intent, or metadata: `Type is irrelevant, but not BOZ (for PRESENT(), OPTIONAL(), &c.)`.
  **L170 CN**: 注释说明附近代码的逻辑、意图或元数据：`Type is irrelevant, but not BOZ (for PRESENT(), OPTIONAL(), &c.)`。
- **L171 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern Addressable{AnyType, KindCode::addressable};`.
  **L171 CN**: 执行一条独立语句或声明：`static constexpr TypePattern Addressable{AnyType, KindCode::addressable};`。
- **L172 EN**: Blank line separating nearby declarations or logic blocks.
  **L172 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L173 EN**: Comment explains nearby logic, intent, or metadata: `Match some kind of some intrinsic type(s); all "Same" values must match,`.
  **L173 CN**: 注释说明附近代码的逻辑、意图或元数据：`Match some kind of some intrinsic type(s); all "Same" values must match,`。
- **L174 EN**: Comment explains nearby logic, intent, or metadata: `even when not in the same category (e.g., SameComplex and SameReal).`.
  **L174 CN**: 注释说明附近代码的逻辑、意图或元数据：`even when not in the same category (e.g., SameComplex and SameReal).`。
- **L175 EN**: Comment explains nearby logic, intent, or metadata: `Can be used to specify a result so long as at least one argument is`.
  **L175 CN**: 注释说明附近代码的逻辑、意图或元数据：`Can be used to specify a result so long as at least one argument is`。
- **L176 EN**: Comment explains nearby logic, intent, or metadata: `a "Same".`.
  **L176 CN**: 注释说明附近代码的逻辑、意图或元数据：`a "Same".`。
- **L177 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameInt{IntType, KindCode::same};`.
  **L177 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameInt{IntType, KindCode::same};`。
- **L178 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern SameIntOrUnsigned{`.
  **L178 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern SameIntOrUnsigned{`。
- **L179 EN**: Executes a standalone statement or declaration: `IntOrUnsignedType, KindCode::same};`.
  **L179 CN**: 执行一条独立语句或声明：`IntOrUnsignedType, KindCode::same};`。
- **L180 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameReal{RealType, KindCode::same};`.
  **L180 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameReal{RealType, KindCode::same};`。
- **L181 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameIntOrReal{IntOrRealType, KindCode::same};`.
  **L181 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameIntOrReal{IntOrRealType, KindCode::same};`。
- **L182 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern SameIntUnsignedOrReal{`.
  **L182 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern SameIntUnsignedOrReal{`。
- **L183 EN**: Executes a standalone statement or declaration: `IntUnsignedOrRealType, KindCode::same};`.
  **L183 CN**: 执行一条独立语句或声明：`IntUnsignedOrRealType, KindCode::same};`。
- **L184 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameComplex{ComplexType, KindCode::same};`.
  **L184 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameComplex{ComplexType, KindCode::same};`。
- **L185 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameFloating{FloatingType, KindCode::same};`.
  **L185 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameFloating{FloatingType, KindCode::same};`。
- **L186 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameNumeric{NumericType, KindCode::same};`.
  **L186 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameNumeric{NumericType, KindCode::same};`。
- **L187 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameChar{CharType, KindCode::same};`.
  **L187 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameChar{CharType, KindCode::same};`。
- **L188 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameCharNoLen{CharType, KindCode::sameKind};`.
  **L188 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameCharNoLen{CharType, KindCode::sameKind};`。
- **L189 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameLogical{LogicalType, KindCode::same};`.
  **L189 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameLogical{LogicalType, KindCode::same};`。
- **L190 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameRelatable{RelatableType, KindCode::same};`.
  **L190 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameRelatable{RelatableType, KindCode::same};`。
- **L191 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameIntrinsic{IntrinsicType, KindCode::same};`.
  **L191 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameIntrinsic{IntrinsicType, KindCode::same};`。
- **L192 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameType{AnyType, KindCode::same};`.
  **L192 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameType{AnyType, KindCode::same};`。

### Lines 193-216

````cpp

// Match some kind of some INTEGER or REAL type(s); when argument types
// &/or kinds differ, their values are converted as if they were operands to
// an intrinsic operation like addition.  This is a nonstandard but nearly
// universal extension feature.
static constexpr TypePattern OperandInt{IntType, KindCode::operand};
static constexpr TypePattern OperandReal{RealType, KindCode::operand};
static constexpr TypePattern OperandIntOrReal{IntOrRealType, KindCode::operand};

static constexpr TypePattern OperandUnsigned{UnsignedType, KindCode::operand};

// For ASSOCIATED, the first argument is a typeless pointer
static constexpr TypePattern AnyPointer{AnyType, KindCode::nullPointerType};

// For DOT_PRODUCT and MATMUL, the result type depends on the arguments
static constexpr TypePattern ResultLogical{LogicalType, KindCode::likeMultiply};
static constexpr TypePattern ResultNumeric{NumericType, KindCode::likeMultiply};

// Result types with known category and KIND=
static constexpr TypePattern KINDInt{IntType, KindCode::effectiveKind};
static constexpr TypePattern KINDUnsigned{
    UnsignedType, KindCode::effectiveKind};
static constexpr TypePattern KINDReal{RealType, KindCode::effectiveKind};
static constexpr TypePattern KINDComplex{ComplexType, KindCode::effectiveKind};
````
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L194 EN**: Comment explains nearby logic, intent, or metadata: `Match some kind of some INTEGER or REAL type(s); when argument types`.
  **L194 CN**: 注释说明附近代码的逻辑、意图或元数据：`Match some kind of some INTEGER or REAL type(s); when argument types`。
- **L195 EN**: Comment explains nearby logic, intent, or metadata: `&/or kinds differ, their values are converted as if they were operands to`.
  **L195 CN**: 注释说明附近代码的逻辑、意图或元数据：`&/or kinds differ, their values are converted as if they were operands to`。
- **L196 EN**: Comment explains nearby logic, intent, or metadata: `an intrinsic operation like addition.  This is a nonstandard but nearly`.
  **L196 CN**: 注释说明附近代码的逻辑、意图或元数据：`an intrinsic operation like addition.  This is a nonstandard but nearly`。
- **L197 EN**: Comment explains nearby logic, intent, or metadata: `universal extension feature.`.
  **L197 CN**: 注释说明附近代码的逻辑、意图或元数据：`universal extension feature.`。
- **L198 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern OperandInt{IntType, KindCode::operand};`.
  **L198 CN**: 执行一条独立语句或声明：`static constexpr TypePattern OperandInt{IntType, KindCode::operand};`。
- **L199 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern OperandReal{RealType, KindCode::operand};`.
  **L199 CN**: 执行一条独立语句或声明：`static constexpr TypePattern OperandReal{RealType, KindCode::operand};`。
- **L200 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern OperandIntOrReal{IntOrRealType, KindCode::operand};`.
  **L200 CN**: 执行一条独立语句或声明：`static constexpr TypePattern OperandIntOrReal{IntOrRealType, KindCode::operand};`。
- **L201 EN**: Blank line separating nearby declarations or logic blocks.
  **L201 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L202 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern OperandUnsigned{UnsignedType, KindCode::operand};`.
  **L202 CN**: 执行一条独立语句或声明：`static constexpr TypePattern OperandUnsigned{UnsignedType, KindCode::operand};`。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L204 EN**: Comment explains nearby logic, intent, or metadata: `For ASSOCIATED, the first argument is a typeless pointer`.
  **L204 CN**: 注释说明附近代码的逻辑、意图或元数据：`For ASSOCIATED, the first argument is a typeless pointer`。
- **L205 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AnyPointer{AnyType, KindCode::nullPointerType};`.
  **L205 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AnyPointer{AnyType, KindCode::nullPointerType};`。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L207 EN**: Comment explains nearby logic, intent, or metadata: `For DOT_PRODUCT and MATMUL, the result type depends on the arguments`.
  **L207 CN**: 注释说明附近代码的逻辑、意图或元数据：`For DOT_PRODUCT and MATMUL, the result type depends on the arguments`。
- **L208 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern ResultLogical{LogicalType, KindCode::likeMultiply};`.
  **L208 CN**: 执行一条独立语句或声明：`static constexpr TypePattern ResultLogical{LogicalType, KindCode::likeMultiply};`。
- **L209 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern ResultNumeric{NumericType, KindCode::likeMultiply};`.
  **L209 CN**: 执行一条独立语句或声明：`static constexpr TypePattern ResultNumeric{NumericType, KindCode::likeMultiply};`。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Comment explains nearby logic, intent, or metadata: `Result types with known category and KIND=`.
  **L211 CN**: 注释说明附近代码的逻辑、意图或元数据：`Result types with known category and KIND=`。
- **L212 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern KINDInt{IntType, KindCode::effectiveKind};`.
  **L212 CN**: 执行一条独立语句或声明：`static constexpr TypePattern KINDInt{IntType, KindCode::effectiveKind};`。
- **L213 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern KINDUnsigned{`.
  **L213 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern KINDUnsigned{`。
- **L214 EN**: Executes a standalone statement or declaration: `UnsignedType, KindCode::effectiveKind};`.
  **L214 CN**: 执行一条独立语句或声明：`UnsignedType, KindCode::effectiveKind};`。
- **L215 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern KINDReal{RealType, KindCode::effectiveKind};`.
  **L215 CN**: 执行一条独立语句或声明：`static constexpr TypePattern KINDReal{RealType, KindCode::effectiveKind};`。
- **L216 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern KINDComplex{ComplexType, KindCode::effectiveKind};`.
  **L216 CN**: 执行一条独立语句或声明：`static constexpr TypePattern KINDComplex{ComplexType, KindCode::effectiveKind};`。

### Lines 217-240

````cpp
static constexpr TypePattern KINDChar{CharType, KindCode::effectiveKind};
static constexpr TypePattern KINDLogical{LogicalType, KindCode::effectiveKind};

static constexpr TypePattern AtomicInt{IntType, KindCode::atomicIntKind};
static constexpr TypePattern AtomicIntOrLogical{
    IntOrLogicalType, KindCode::atomicIntOrLogicalKind};
static constexpr TypePattern SameAtom{IntOrLogicalType, KindCode::sameAtom};

// The default rank pattern for dummy arguments and function results is
// "elemental".
ENUM_CLASS(Rank,
    elemental, // scalar, or array that conforms with other array arguments
    elementalOrBOZ, // elemental, or typeless BOZ literal scalar
    scalar, vector,
    shape, // INTEGER vector of known length and no negative element
    matrix,
    array, // not scalar, rank is known and greater than zero
    coarray, // rank is known and can be scalar; has nonzero corank
    atom, // is scalar and has nonzero corank or is coindexed
    known, // rank is known and can be scalar
    anyOrAssumedRank, // any rank, or assumed; assumed-type TYPE(*) allowed
    arrayOrAssumedRank, // rank >= 1 or assumed; assumed-type TYPE(*) allowed
    conformable, // scalar, or array of same rank & shape as "array" argument
    reduceOperation, // a pure function with constraints for REDUCE
````
- **L217 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern KINDChar{CharType, KindCode::effectiveKind};`.
  **L217 CN**: 执行一条独立语句或声明：`static constexpr TypePattern KINDChar{CharType, KindCode::effectiveKind};`。
- **L218 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern KINDLogical{LogicalType, KindCode::effectiveKind};`.
  **L218 CN**: 执行一条独立语句或声明：`static constexpr TypePattern KINDLogical{LogicalType, KindCode::effectiveKind};`。
- **L219 EN**: Blank line separating nearby declarations or logic blocks.
  **L219 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L220 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern AtomicInt{IntType, KindCode::atomicIntKind};`.
  **L220 CN**: 执行一条独立语句或声明：`static constexpr TypePattern AtomicInt{IntType, KindCode::atomicIntKind};`。
- **L221 EN**: Continues the surrounding expression or declaration: `static constexpr TypePattern AtomicIntOrLogical{`.
  **L221 CN**: 继续构造周围的表达式或声明：`static constexpr TypePattern AtomicIntOrLogical{`。
- **L222 EN**: Executes a standalone statement or declaration: `IntOrLogicalType, KindCode::atomicIntOrLogicalKind};`.
  **L222 CN**: 执行一条独立语句或声明：`IntOrLogicalType, KindCode::atomicIntOrLogicalKind};`。
- **L223 EN**: Executes a standalone statement or declaration: `static constexpr TypePattern SameAtom{IntOrLogicalType, KindCode::sameAtom};`.
  **L223 CN**: 执行一条独立语句或声明：`static constexpr TypePattern SameAtom{IntOrLogicalType, KindCode::sameAtom};`。
- **L224 EN**: Blank line separating nearby declarations or logic blocks.
  **L224 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L225 EN**: Comment explains nearby logic, intent, or metadata: `The default rank pattern for dummy arguments and function results is`.
  **L225 CN**: 注释说明附近代码的逻辑、意图或元数据：`The default rank pattern for dummy arguments and function results is`。
- **L226 EN**: Comment explains nearby logic, intent, or metadata: `"elemental".`.
  **L226 CN**: 注释说明附近代码的逻辑、意图或元数据：`"elemental".`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ENUM_CLASS(Rank,`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`ENUM_CLASS(Rank,`。
- **L228 EN**: Continues the surrounding expression or declaration: `elemental, // scalar, or array that conforms with other array arguments`.
  **L228 CN**: 继续构造周围的表达式或声明：`elemental, // scalar, or array that conforms with other array arguments`。
- **L229 EN**: Continues the surrounding expression or declaration: `elementalOrBOZ, // elemental, or typeless BOZ literal scalar`.
  **L229 CN**: 继续构造周围的表达式或声明：`elementalOrBOZ, // elemental, or typeless BOZ literal scalar`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `scalar, vector,`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`scalar, vector,`。
- **L231 EN**: Continues the surrounding expression or declaration: `shape, // INTEGER vector of known length and no negative element`.
  **L231 CN**: 继续构造周围的表达式或声明：`shape, // INTEGER vector of known length and no negative element`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matrix,`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`matrix,`。
- **L233 EN**: Continues the surrounding expression or declaration: `array, // not scalar, rank is known and greater than zero`.
  **L233 CN**: 继续构造周围的表达式或声明：`array, // not scalar, rank is known and greater than zero`。
- **L234 EN**: Continues the surrounding expression or declaration: `coarray, // rank is known and can be scalar; has nonzero corank`.
  **L234 CN**: 继续构造周围的表达式或声明：`coarray, // rank is known and can be scalar; has nonzero corank`。
- **L235 EN**: Continues the surrounding expression or declaration: `atom, // is scalar and has nonzero corank or is coindexed`.
  **L235 CN**: 继续构造周围的表达式或声明：`atom, // is scalar and has nonzero corank or is coindexed`。
- **L236 EN**: Continues the surrounding expression or declaration: `known, // rank is known and can be scalar`.
  **L236 CN**: 继续构造周围的表达式或声明：`known, // rank is known and can be scalar`。
- **L237 EN**: Continues logic associated with callable symbol `TYPE`.
  **L237 CN**: 继续与可调用符号 `TYPE` 相关的逻辑。
- **L238 EN**: Continues logic associated with callable symbol `TYPE`.
  **L238 CN**: 继续与可调用符号 `TYPE` 相关的逻辑。
- **L239 EN**: Continues the surrounding expression or declaration: `conformable, // scalar, or array of same rank & shape as "array" argument`.
  **L239 CN**: 继续构造周围的表达式或声明：`conformable, // scalar, or array of same rank & shape as "array" argument`。
- **L240 EN**: Continues the surrounding expression or declaration: `reduceOperation, // a pure function with constraints for REDUCE`.
  **L240 CN**: 继续构造周围的表达式或声明：`reduceOperation, // a pure function with constraints for REDUCE`。

### Lines 241-264

````cpp
    dimReduced, // scalar if no DIM= argument, else rank(array)-1
    dimRemovedOrScalar, // rank(array)-1 (less DIM) or scalar
    scalarIfDim, // scalar if DIM= argument is present, else rank one array
    locReduced, // vector(1:rank) if no DIM= argument, else rank(array)-1
    rankPlus1, // rank(known)+1
    shaped, // rank is length of SHAPE vector
)

ENUM_CLASS(Optionality, required,
    optional, // unless DIM= for SIZE(assumedSize)
    missing, // for DIM= cases like FINDLOC
    repeats, // for MAX/MIN and their several variants
)

ENUM_CLASS(ArgFlag, none,
    canBeNullPointer, // actual argument can be NULL(with or without
                      // MOLD=pointer)
    canBeMoldNull, // actual argument can be NULL(MOLD=any)
    canBeNullAllocatable, // actual argument can be NULL(MOLD=allocatable)
    defaultsToSameKind, // for MatchingDefaultKIND
    defaultsToSizeKind, // for SizeDefaultKIND
    defaultsToDefaultForResult, // for DefaultingKIND
    notAssumedSize,
    onlyConstantInquiry) // e.g., PRECISION(X)
````
- **L241 EN**: Continues logic associated with callable symbol `rank`.
  **L241 CN**: 继续与可调用符号 `rank` 相关的逻辑。
- **L242 EN**: Continues logic associated with callable symbol `rank`.
  **L242 CN**: 继续与可调用符号 `rank` 相关的逻辑。
- **L243 EN**: Continues the surrounding expression or declaration: `scalarIfDim, // scalar if DIM= argument is present, else rank one array`.
  **L243 CN**: 继续构造周围的表达式或声明：`scalarIfDim, // scalar if DIM= argument is present, else rank one array`。
- **L244 EN**: Continues logic associated with callable symbol `vector`.
  **L244 CN**: 继续与可调用符号 `vector` 相关的逻辑。
- **L245 EN**: Continues logic associated with callable symbol `rank`.
  **L245 CN**: 继续与可调用符号 `rank` 相关的逻辑。
- **L246 EN**: Continues the surrounding expression or declaration: `shaped, // rank is length of SHAPE vector`.
  **L246 CN**: 继续构造周围的表达式或声明：`shaped, // rank is length of SHAPE vector`。
- **L247 EN**: Continues the surrounding expression or declaration: `)`.
  **L247 CN**: 继续构造周围的表达式或声明：`)`。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ENUM_CLASS(Optionality, required,`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`ENUM_CLASS(Optionality, required,`。
- **L250 EN**: Continues logic associated with callable symbol `SIZE`.
  **L250 CN**: 继续与可调用符号 `SIZE` 相关的逻辑。
- **L251 EN**: Continues the surrounding expression or declaration: `missing, // for DIM= cases like FINDLOC`.
  **L251 CN**: 继续构造周围的表达式或声明：`missing, // for DIM= cases like FINDLOC`。
- **L252 EN**: Continues the surrounding expression or declaration: `repeats, // for MAX/MIN and their several variants`.
  **L252 CN**: 继续构造周围的表达式或声明：`repeats, // for MAX/MIN and their several variants`。
- **L253 EN**: Continues the surrounding expression or declaration: `)`.
  **L253 CN**: 继续构造周围的表达式或声明：`)`。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ENUM_CLASS(ArgFlag, none,`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`ENUM_CLASS(ArgFlag, none,`。
- **L256 EN**: Continues logic associated with callable symbol `NULL`.
  **L256 CN**: 继续与可调用符号 `NULL` 相关的逻辑。
- **L257 EN**: Comment explains nearby logic, intent, or metadata: `MOLD=pointer)`.
  **L257 CN**: 注释说明附近代码的逻辑、意图或元数据：`MOLD=pointer)`。
- **L258 EN**: Continues logic associated with callable symbol `NULL`.
  **L258 CN**: 继续与可调用符号 `NULL` 相关的逻辑。
- **L259 EN**: Continues logic associated with callable symbol `NULL`.
  **L259 CN**: 继续与可调用符号 `NULL` 相关的逻辑。
- **L260 EN**: Continues the surrounding expression or declaration: `defaultsToSameKind, // for MatchingDefaultKIND`.
  **L260 CN**: 继续构造周围的表达式或声明：`defaultsToSameKind, // for MatchingDefaultKIND`。
- **L261 EN**: Continues the surrounding expression or declaration: `defaultsToSizeKind, // for SizeDefaultKIND`.
  **L261 CN**: 继续构造周围的表达式或声明：`defaultsToSizeKind, // for SizeDefaultKIND`。
- **L262 EN**: Continues the surrounding expression or declaration: `defaultsToDefaultForResult, // for DefaultingKIND`.
  **L262 CN**: 继续构造周围的表达式或声明：`defaultsToDefaultForResult, // for DefaultingKIND`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `notAssumedSize,`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`notAssumedSize,`。
- **L264 EN**: Continues logic associated with callable symbol `PRECISION`.
  **L264 CN**: 继续与可调用符号 `PRECISION` 相关的逻辑。

### Lines 265-288

````cpp

struct IntrinsicDummyArgument {
  const char *keyword{nullptr};
  TypePattern typePattern;
  Rank rank{Rank::elemental};
  Optionality optionality{Optionality::required};
  common::Intent intent{common::Intent::In};
  common::EnumSet<ArgFlag, 32> flags{};
  llvm::raw_ostream &Dump(llvm::raw_ostream &) const;
};

// constexpr abbreviations for popular arguments:
// DefaultingKIND is a KIND= argument whose default value is the appropriate
// KIND(0), KIND(0.0), KIND(''), &c. value for the function result.
static constexpr IntrinsicDummyArgument DefaultingKIND{"kind",
    {IntType, KindCode::kindArg}, Rank::scalar, Optionality::optional,
    common::Intent::In, {ArgFlag::defaultsToDefaultForResult}};
// MatchingDefaultKIND is a KIND= argument whose default value is the
// kind of any "Same" function argument (viz., the one whose kind pattern is
// "same").
static constexpr IntrinsicDummyArgument MatchingDefaultKIND{"kind",
    {IntType, KindCode::kindArg}, Rank::scalar, Optionality::optional,
    common::Intent::In, {ArgFlag::defaultsToSameKind}};
// SizeDefaultKind is a KIND= argument whose default value should be
````
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Declares struct `IntrinsicDummyArgument`.
  **L266 CN**: 声明 struct `IntrinsicDummyArgument`。
- **L267 EN**: Executes a standalone statement or declaration: `const char *keyword{nullptr};`.
  **L267 CN**: 执行一条独立语句或声明：`const char *keyword{nullptr};`。
- **L268 EN**: Executes a standalone statement or declaration: `TypePattern typePattern;`.
  **L268 CN**: 执行一条独立语句或声明：`TypePattern typePattern;`。
- **L269 EN**: Executes a standalone statement or declaration: `Rank rank{Rank::elemental};`.
  **L269 CN**: 执行一条独立语句或声明：`Rank rank{Rank::elemental};`。
- **L270 EN**: Executes a standalone statement or declaration: `Optionality optionality{Optionality::required};`.
  **L270 CN**: 执行一条独立语句或声明：`Optionality optionality{Optionality::required};`。
- **L271 EN**: Executes a standalone statement or declaration: `common::Intent intent{common::Intent::In};`.
  **L271 CN**: 执行一条独立语句或声明：`common::Intent intent{common::Intent::In};`。
- **L272 EN**: Executes a standalone statement or declaration: `common::EnumSet<ArgFlag, 32> flags{};`.
  **L272 CN**: 执行一条独立语句或声明：`common::EnumSet<ArgFlag, 32> flags{};`。
- **L273 EN**: Executes a call or declaration centered on `&Dump`.
  **L273 CN**: 执行以 `&Dump` 为核心的调用或声明。
- **L274 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L274 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L275 EN**: Blank line separating nearby declarations or logic blocks.
  **L275 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L276 EN**: Comment explains nearby logic, intent, or metadata: `constexpr abbreviations for popular arguments:`.
  **L276 CN**: 注释说明附近代码的逻辑、意图或元数据：`constexpr abbreviations for popular arguments:`。
- **L277 EN**: Comment explains nearby logic, intent, or metadata: `DefaultingKIND is a KIND= argument whose default value is the appropriate`.
  **L277 CN**: 注释说明附近代码的逻辑、意图或元数据：`DefaultingKIND is a KIND= argument whose default value is the appropriate`。
- **L278 EN**: Comment explains nearby logic, intent, or metadata: `KIND(0), KIND(0.0), KIND(''), &c. value for the function result.`.
  **L278 CN**: 注释说明附近代码的逻辑、意图或元数据：`KIND(0), KIND(0.0), KIND(''), &c. value for the function result.`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static constexpr IntrinsicDummyArgument DefaultingKIND{"kind",`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`static constexpr IntrinsicDummyArgument DefaultingKIND{"kind",`。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IntType, KindCode::kindArg}, Rank::scalar, Optionality::optional,`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IntType, KindCode::kindArg}, Rank::scalar, Optionality::optional,`。
- **L281 EN**: Executes a standalone statement or declaration: `common::Intent::In, {ArgFlag::defaultsToDefaultForResult}};`.
  **L281 CN**: 执行一条独立语句或声明：`common::Intent::In, {ArgFlag::defaultsToDefaultForResult}};`。
- **L282 EN**: Comment explains nearby logic, intent, or metadata: `MatchingDefaultKIND is a KIND= argument whose default value is the`.
  **L282 CN**: 注释说明附近代码的逻辑、意图或元数据：`MatchingDefaultKIND is a KIND= argument whose default value is the`。
- **L283 EN**: Comment explains nearby logic, intent, or metadata: `kind of any "Same" function argument (viz., the one whose kind pattern is`.
  **L283 CN**: 注释说明附近代码的逻辑、意图或元数据：`kind of any "Same" function argument (viz., the one whose kind pattern is`。
- **L284 EN**: Comment explains nearby logic, intent, or metadata: `"same").`.
  **L284 CN**: 注释说明附近代码的逻辑、意图或元数据：`"same").`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static constexpr IntrinsicDummyArgument MatchingDefaultKIND{"kind",`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`static constexpr IntrinsicDummyArgument MatchingDefaultKIND{"kind",`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IntType, KindCode::kindArg}, Rank::scalar, Optionality::optional,`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IntType, KindCode::kindArg}, Rank::scalar, Optionality::optional,`。
- **L287 EN**: Executes a standalone statement or declaration: `common::Intent::In, {ArgFlag::defaultsToSameKind}};`.
  **L287 CN**: 执行一条独立语句或声明：`common::Intent::In, {ArgFlag::defaultsToSameKind}};`。
- **L288 EN**: Comment explains nearby logic, intent, or metadata: `SizeDefaultKind is a KIND= argument whose default value should be`.
  **L288 CN**: 注释说明附近代码的逻辑、意图或元数据：`SizeDefaultKind is a KIND= argument whose default value should be`。

### Lines 289-312

````cpp
// the kind of INTEGER used for address calculations, and can be
// set so with a compiler flag; but the standard mandates the
// kind of default INTEGER.
static constexpr IntrinsicDummyArgument SizeDefaultKIND{"kind",
    {IntType, KindCode::kindArg}, Rank::scalar, Optionality::optional,
    common::Intent::In, {ArgFlag::defaultsToSizeKind}};
static constexpr IntrinsicDummyArgument RequiredDIM{"dim",
    {IntType, KindCode::dimArg}, Rank::scalar, Optionality::required,
    common::Intent::In};
static constexpr IntrinsicDummyArgument OptionalDIM{"dim",
    {IntType, KindCode::dimArg}, Rank::scalar, Optionality::optional,
    common::Intent::In};
static constexpr IntrinsicDummyArgument MissingDIM{"dim",
    {IntType, KindCode::dimArg}, Rank::scalar, Optionality::missing,
    common::Intent::In};
static constexpr IntrinsicDummyArgument OptionalMASK{"mask", AnyLogical,
    Rank::conformable, Optionality::optional, common::Intent::In};
static constexpr IntrinsicDummyArgument OptionalTEAM{
    "team", TeamType, Rank::scalar, Optionality::optional, common::Intent::In};

struct IntrinsicInterface {
  static constexpr int maxArguments{7}; // if not a MAX/MIN(...)
  const char *name{nullptr};
  IntrinsicDummyArgument dummy[maxArguments];
````
- **L289 EN**: Comment explains nearby logic, intent, or metadata: `the kind of INTEGER used for address calculations, and can be`.
  **L289 CN**: 注释说明附近代码的逻辑、意图或元数据：`the kind of INTEGER used for address calculations, and can be`。
- **L290 EN**: Comment explains nearby logic, intent, or metadata: `set so with a compiler flag; but the standard mandates the`.
  **L290 CN**: 注释说明附近代码的逻辑、意图或元数据：`set so with a compiler flag; but the standard mandates the`。
- **L291 EN**: Comment explains nearby logic, intent, or metadata: `kind of default INTEGER.`.
  **L291 CN**: 注释说明附近代码的逻辑、意图或元数据：`kind of default INTEGER.`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static constexpr IntrinsicDummyArgument SizeDefaultKIND{"kind",`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`static constexpr IntrinsicDummyArgument SizeDefaultKIND{"kind",`。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IntType, KindCode::kindArg}, Rank::scalar, Optionality::optional,`.
  **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IntType, KindCode::kindArg}, Rank::scalar, Optionality::optional,`。
- **L294 EN**: Executes a standalone statement or declaration: `common::Intent::In, {ArgFlag::defaultsToSizeKind}};`.
  **L294 CN**: 执行一条独立语句或声明：`common::Intent::In, {ArgFlag::defaultsToSizeKind}};`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static constexpr IntrinsicDummyArgument RequiredDIM{"dim",`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`static constexpr IntrinsicDummyArgument RequiredDIM{"dim",`。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IntType, KindCode::dimArg}, Rank::scalar, Optionality::required,`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IntType, KindCode::dimArg}, Rank::scalar, Optionality::required,`。
- **L297 EN**: Executes a standalone statement or declaration: `common::Intent::In};`.
  **L297 CN**: 执行一条独立语句或声明：`common::Intent::In};`。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static constexpr IntrinsicDummyArgument OptionalDIM{"dim",`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`static constexpr IntrinsicDummyArgument OptionalDIM{"dim",`。
- **L299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IntType, KindCode::dimArg}, Rank::scalar, Optionality::optional,`.
  **L299 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IntType, KindCode::dimArg}, Rank::scalar, Optionality::optional,`。
- **L300 EN**: Executes a standalone statement or declaration: `common::Intent::In};`.
  **L300 CN**: 执行一条独立语句或声明：`common::Intent::In};`。
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static constexpr IntrinsicDummyArgument MissingDIM{"dim",`.
  **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`static constexpr IntrinsicDummyArgument MissingDIM{"dim",`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IntType, KindCode::dimArg}, Rank::scalar, Optionality::missing,`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IntType, KindCode::dimArg}, Rank::scalar, Optionality::missing,`。
- **L303 EN**: Executes a standalone statement or declaration: `common::Intent::In};`.
  **L303 CN**: 执行一条独立语句或声明：`common::Intent::In};`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static constexpr IntrinsicDummyArgument OptionalMASK{"mask", AnyLogical,`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`static constexpr IntrinsicDummyArgument OptionalMASK{"mask", AnyLogical,`。
- **L305 EN**: Executes a standalone statement or declaration: `Rank::conformable, Optionality::optional, common::Intent::In};`.
  **L305 CN**: 执行一条独立语句或声明：`Rank::conformable, Optionality::optional, common::Intent::In};`。
- **L306 EN**: Continues the surrounding expression or declaration: `static constexpr IntrinsicDummyArgument OptionalTEAM{`.
  **L306 CN**: 继续构造周围的表达式或声明：`static constexpr IntrinsicDummyArgument OptionalTEAM{`。
- **L307 EN**: Executes a standalone statement or declaration: `"team", TeamType, Rank::scalar, Optionality::optional, common::Intent::In};`.
  **L307 CN**: 执行一条独立语句或声明：`"team", TeamType, Rank::scalar, Optionality::optional, common::Intent::In};`。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L309 EN**: Declares struct `IntrinsicInterface`.
  **L309 CN**: 声明 struct `IntrinsicInterface`。
- **L310 EN**: Continues logic associated with callable symbol `MIN`.
  **L310 CN**: 继续与可调用符号 `MIN` 相关的逻辑。
- **L311 EN**: Executes a standalone statement or declaration: `const char *name{nullptr};`.
  **L311 CN**: 执行一条独立语句或声明：`const char *name{nullptr};`。
- **L312 EN**: Executes a standalone statement or declaration: `IntrinsicDummyArgument dummy[maxArguments];`.
  **L312 CN**: 执行一条独立语句或声明：`IntrinsicDummyArgument dummy[maxArguments];`。

### Lines 313-336

````cpp
  TypePattern result;
  Rank rank{Rank::elemental};
  IntrinsicClass intrinsicClass{IntrinsicClass::elementalFunction};
  std::optional<SpecificCall> Match(const CallCharacteristics &,
      const common::IntrinsicTypeDefaultKinds &, ActualArguments &,
      FoldingContext &context, const semantics::Scope *builtins) const;
  int CountArguments() const;
  llvm::raw_ostream &Dump(llvm::raw_ostream &) const;
};

int IntrinsicInterface::CountArguments() const {
  int n{0};
  while (n < maxArguments && dummy[n].keyword) {
    ++n;
  }
  return n;
}

// GENERIC INTRINSIC FUNCTION INTERFACES
// Each entry in this table defines a pattern.  Some intrinsic
// functions have more than one such pattern.  Besides the name
// of the intrinsic function, each pattern has specifications for
// the dummy arguments and for the result of the function.
// The dummy argument patterns each have a name (these are from the
````
- **L313 EN**: Executes a standalone statement or declaration: `TypePattern result;`.
  **L313 CN**: 执行一条独立语句或声明：`TypePattern result;`。
- **L314 EN**: Executes a standalone statement or declaration: `Rank rank{Rank::elemental};`.
  **L314 CN**: 执行一条独立语句或声明：`Rank rank{Rank::elemental};`。
- **L315 EN**: Executes a standalone statement or declaration: `IntrinsicClass intrinsicClass{IntrinsicClass::elementalFunction};`.
  **L315 CN**: 执行一条独立语句或声明：`IntrinsicClass intrinsicClass{IntrinsicClass::elementalFunction};`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<SpecificCall> Match(const CallCharacteristics &,`.
  **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<SpecificCall> Match(const CallCharacteristics &,`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const common::IntrinsicTypeDefaultKinds &, ActualArguments &,`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`const common::IntrinsicTypeDefaultKinds &, ActualArguments &,`。
- **L318 EN**: Executes a standalone statement or declaration: `FoldingContext &context, const semantics::Scope *builtins) const;`.
  **L318 CN**: 执行一条独立语句或声明：`FoldingContext &context, const semantics::Scope *builtins) const;`。
- **L319 EN**: Executes a call or declaration centered on `CountArguments`.
  **L319 CN**: 执行以 `CountArguments` 为核心的调用或声明。
- **L320 EN**: Executes a call or declaration centered on `&Dump`.
  **L320 CN**: 执行以 `&Dump` 为核心的调用或声明。
- **L321 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L321 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L323 EN**: Starts a function, method, lambda, or structured scope: `int IntrinsicInterface::CountArguments() const {`.
  **L323 CN**: 开始一个函数、方法、lambda 或结构化作用域：`int IntrinsicInterface::CountArguments() const {`。
- **L324 EN**: Executes a standalone statement or declaration: `int n{0};`.
  **L324 CN**: 执行一条独立语句或声明：`int n{0};`。
- **L325 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L325 CN**: 开始 `while` 控制流语句并计算其条件。
- **L326 EN**: Executes a standalone statement or declaration: `++n;`.
  **L326 CN**: 执行一条独立语句或声明：`++n;`。
- **L327 EN**: Closes the current lexical scope or compound statement.
  **L327 CN**: 结束当前词法作用域或复合语句块。
- **L328 EN**: Returns from the current function with `n`.
  **L328 CN**: 以 `n` 从当前函数返回。
- **L329 EN**: Closes the current lexical scope or compound statement.
  **L329 CN**: 结束当前词法作用域或复合语句块。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L331 EN**: Comment explains nearby logic, intent, or metadata: `GENERIC INTRINSIC FUNCTION INTERFACES`.
  **L331 CN**: 注释说明附近代码的逻辑、意图或元数据：`GENERIC INTRINSIC FUNCTION INTERFACES`。
- **L332 EN**: Comment explains nearby logic, intent, or metadata: `Each entry in this table defines a pattern.  Some intrinsic`.
  **L332 CN**: 注释说明附近代码的逻辑、意图或元数据：`Each entry in this table defines a pattern.  Some intrinsic`。
- **L333 EN**: Comment explains nearby logic, intent, or metadata: `functions have more than one such pattern.  Besides the name`.
  **L333 CN**: 注释说明附近代码的逻辑、意图或元数据：`functions have more than one such pattern.  Besides the name`。
- **L334 EN**: Comment explains nearby logic, intent, or metadata: `of the intrinsic function, each pattern has specifications for`.
  **L334 CN**: 注释说明附近代码的逻辑、意图或元数据：`of the intrinsic function, each pattern has specifications for`。
- **L335 EN**: Comment explains nearby logic, intent, or metadata: `the dummy arguments and for the result of the function.`.
  **L335 CN**: 注释说明附近代码的逻辑、意图或元数据：`the dummy arguments and for the result of the function.`。
- **L336 EN**: Comment explains nearby logic, intent, or metadata: `The dummy argument patterns each have a name (these are from the`.
  **L336 CN**: 注释说明附近代码的逻辑、意图或元数据：`The dummy argument patterns each have a name (these are from the`。

### Lines 337-360

````cpp
// standard, but rarely appear in actual code), a type and kind
// pattern, allowable ranks, and optionality indicators.
// Be advised, the default rank pattern is "elemental".
static const IntrinsicInterface genericIntrinsicFunction[]{
    {"abs", {{"a", SameIntOrReal}}, SameIntOrReal},
    {"abs", {{"a", SameComplex}}, SameReal},
    {"achar", {{"i", AnyInt, Rank::elementalOrBOZ}, DefaultingKIND}, KINDChar},
    {"acos", {{"x", SameFloating}}, SameFloating},
    {"acosd", {{"x", SameFloating}}, SameFloating},
    {"acosh", {{"x", SameFloating}}, SameFloating},
    {"acospi", {{"x", SameFloating}}, SameFloating},
    {"adjustl", {{"string", SameChar}}, SameChar},
    {"adjustr", {{"string", SameChar}}, SameChar},
    {"aimag", {{"z", SameComplex}}, SameReal},
    {"aint", {{"a", SameReal}, MatchingDefaultKIND}, KINDReal},
    {"all", {{"mask", SameLogical, Rank::array}, OptionalDIM}, SameLogical,
        Rank::dimReduced, IntrinsicClass::transformationalFunction},
    {"allocated", {{"scalar", AnyData, Rank::scalar}}, DefaultLogical,
        Rank::elemental, IntrinsicClass::inquiryFunction},
    {"allocated",
        {{"array", AnyData, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In, {ArgFlag::canBeNullAllocatable}}},
        DefaultLogical, Rank::elemental, IntrinsicClass::inquiryFunction},
    {"anint", {{"a", SameReal}, MatchingDefaultKIND}, KINDReal},
````
- **L337 EN**: Comment explains nearby logic, intent, or metadata: `standard, but rarely appear in actual code), a type and kind`.
  **L337 CN**: 注释说明附近代码的逻辑、意图或元数据：`standard, but rarely appear in actual code), a type and kind`。
- **L338 EN**: Comment explains nearby logic, intent, or metadata: `pattern, allowable ranks, and optionality indicators.`.
  **L338 CN**: 注释说明附近代码的逻辑、意图或元数据：`pattern, allowable ranks, and optionality indicators.`。
- **L339 EN**: Comment explains nearby logic, intent, or metadata: `Be advised, the default rank pattern is "elemental".`.
  **L339 CN**: 注释说明附近代码的逻辑、意图或元数据：`Be advised, the default rank pattern is "elemental".`。
- **L340 EN**: Continues the surrounding expression or declaration: `static const IntrinsicInterface genericIntrinsicFunction[]{`.
  **L340 CN**: 继续构造周围的表达式或声明：`static const IntrinsicInterface genericIntrinsicFunction[]{`。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"abs", {{"a", SameIntOrReal}}, SameIntOrReal},`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"abs", {{"a", SameIntOrReal}}, SameIntOrReal},`。
- **L342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"abs", {{"a", SameComplex}}, SameReal},`.
  **L342 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"abs", {{"a", SameComplex}}, SameReal},`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"achar", {{"i", AnyInt, Rank::elementalOrBOZ}, DefaultingKIND}, KINDChar},`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"achar", {{"i", AnyInt, Rank::elementalOrBOZ}, DefaultingKIND}, KINDChar},`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"acos", {{"x", SameFloating}}, SameFloating},`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"acos", {{"x", SameFloating}}, SameFloating},`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"acosd", {{"x", SameFloating}}, SameFloating},`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"acosd", {{"x", SameFloating}}, SameFloating},`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"acosh", {{"x", SameFloating}}, SameFloating},`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"acosh", {{"x", SameFloating}}, SameFloating},`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"acospi", {{"x", SameFloating}}, SameFloating},`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"acospi", {{"x", SameFloating}}, SameFloating},`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"adjustl", {{"string", SameChar}}, SameChar},`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"adjustl", {{"string", SameChar}}, SameChar},`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"adjustr", {{"string", SameChar}}, SameChar},`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"adjustr", {{"string", SameChar}}, SameChar},`。
- **L350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"aimag", {{"z", SameComplex}}, SameReal},`.
  **L350 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"aimag", {{"z", SameComplex}}, SameReal},`。
- **L351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"aint", {{"a", SameReal}, MatchingDefaultKIND}, KINDReal},`.
  **L351 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"aint", {{"a", SameReal}, MatchingDefaultKIND}, KINDReal},`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"all", {{"mask", SameLogical, Rank::array}, OptionalDIM}, SameLogical,`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"all", {{"mask", SameLogical, Rank::array}, OptionalDIM}, SameLogical,`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::dimReduced, IntrinsicClass::transformationalFunction},`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::dimReduced, IntrinsicClass::transformationalFunction},`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"allocated", {{"scalar", AnyData, Rank::scalar}}, DefaultLogical,`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"allocated", {{"scalar", AnyData, Rank::scalar}}, DefaultLogical,`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::elemental, IntrinsicClass::inquiryFunction},`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::elemental, IntrinsicClass::inquiryFunction},`。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"allocated",`.
  **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"allocated",`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyData, Rank::anyOrAssumedRank, Optionality::required,`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyData, Rank::anyOrAssumedRank, Optionality::required,`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In, {ArgFlag::canBeNullAllocatable}}},`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In, {ArgFlag::canBeNullAllocatable}}},`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical, Rank::elemental, IntrinsicClass::inquiryFunction},`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical, Rank::elemental, IntrinsicClass::inquiryFunction},`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"anint", {{"a", SameReal}, MatchingDefaultKIND}, KINDReal},`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"anint", {{"a", SameReal}, MatchingDefaultKIND}, KINDReal},`。

### Lines 361-384

````cpp
    {"any", {{"mask", SameLogical, Rank::array}, OptionalDIM}, SameLogical,
        Rank::dimReduced, IntrinsicClass::transformationalFunction},
    {"asin", {{"x", SameFloating}}, SameFloating},
    {"asind", {{"x", SameFloating}}, SameFloating},
    {"asinh", {{"x", SameFloating}}, SameFloating},
    {"asinpi", {{"x", SameFloating}}, SameFloating},
    {"associated",
        {{"pointer", AnyPointer, Rank::anyOrAssumedRank, Optionality::required,
             common::Intent::In, {ArgFlag::canBeNullPointer}},
            {"target", Addressable, Rank::anyOrAssumedRank,
                Optionality::optional, common::Intent::In,
                {ArgFlag::canBeNullPointer}}},
        DefaultLogical, Rank::elemental, IntrinsicClass::inquiryFunction},
    {"atan", {{"x", SameFloating}}, SameFloating},
    {"atan", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},
    {"atand", {{"x", SameFloating}}, SameFloating},
    {"atand", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},
    {"atan2", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},
    {"atan2d", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},
    {"atanpi", {{"x", SameFloating}}, SameFloating},
    {"atanpi", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},
    {"atan2pi", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},
    {"atanh", {{"x", SameFloating}}, SameFloating},
    {"bessel_j0", {{"x", SameReal}}, SameReal},
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"any", {{"mask", SameLogical, Rank::array}, OptionalDIM}, SameLogical,`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"any", {{"mask", SameLogical, Rank::array}, OptionalDIM}, SameLogical,`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::dimReduced, IntrinsicClass::transformationalFunction},`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::dimReduced, IntrinsicClass::transformationalFunction},`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"asin", {{"x", SameFloating}}, SameFloating},`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"asin", {{"x", SameFloating}}, SameFloating},`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"asind", {{"x", SameFloating}}, SameFloating},`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"asind", {{"x", SameFloating}}, SameFloating},`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"asinh", {{"x", SameFloating}}, SameFloating},`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"asinh", {{"x", SameFloating}}, SameFloating},`。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"asinpi", {{"x", SameFloating}}, SameFloating},`.
  **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"asinpi", {{"x", SameFloating}}, SameFloating},`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"associated",`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"associated",`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"pointer", AnyPointer, Rank::anyOrAssumedRank, Optionality::required,`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"pointer", AnyPointer, Rank::anyOrAssumedRank, Optionality::required,`。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In, {ArgFlag::canBeNullPointer}},`.
  **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In, {ArgFlag::canBeNullPointer}},`。
- **L370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"target", Addressable, Rank::anyOrAssumedRank,`.
  **L370 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"target", Addressable, Rank::anyOrAssumedRank,`。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::optional, common::Intent::In,`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::optional, common::Intent::In,`。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeNullPointer}}},`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeNullPointer}}},`。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical, Rank::elemental, IntrinsicClass::inquiryFunction},`.
  **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical, Rank::elemental, IntrinsicClass::inquiryFunction},`。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atan", {{"x", SameFloating}}, SameFloating},`.
  **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atan", {{"x", SameFloating}}, SameFloating},`。
- **L375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atan", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`.
  **L375 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atan", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`。
- **L376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atand", {{"x", SameFloating}}, SameFloating},`.
  **L376 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atand", {{"x", SameFloating}}, SameFloating},`。
- **L377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atand", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`.
  **L377 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atand", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`。
- **L378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atan2", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`.
  **L378 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atan2", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`。
- **L379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atan2d", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`.
  **L379 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atan2d", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`。
- **L380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atanpi", {{"x", SameFloating}}, SameFloating},`.
  **L380 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atanpi", {{"x", SameFloating}}, SameFloating},`。
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atanpi", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atanpi", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atan2pi", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atan2pi", {{"y", OperandReal}, {"x", OperandReal}}, OperandReal},`。
- **L383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atanh", {{"x", SameFloating}}, SameFloating},`.
  **L383 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atanh", {{"x", SameFloating}}, SameFloating},`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bessel_j0", {{"x", SameReal}}, SameReal},`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bessel_j0", {{"x", SameReal}}, SameReal},`。

### Lines 385-408

````cpp
    {"bessel_j1", {{"x", SameReal}}, SameReal},
    {"bessel_jn", {{"n", AnyInt}, {"x", SameReal}}, SameReal},
    {"bessel_jn",
        {{"n1", AnyInt, Rank::scalar}, {"n2", AnyInt, Rank::scalar},
            {"x", SameReal, Rank::scalar}},
        SameReal, Rank::vector, IntrinsicClass::transformationalFunction},
    {"bessel_y0", {{"x", SameReal}}, SameReal},
    {"bessel_y1", {{"x", SameReal}}, SameReal},
    {"bessel_yn", {{"n", AnyInt}, {"x", SameReal}}, SameReal},
    {"bessel_yn",
        {{"n1", AnyInt, Rank::scalar}, {"n2", AnyInt, Rank::scalar},
            {"x", SameReal, Rank::scalar}},
        SameReal, Rank::vector, IntrinsicClass::transformationalFunction},
    {"bge",
        {{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},
            {"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},
        DefaultLogical},
    {"bgt",
        {{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},
            {"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},
        DefaultLogical},
    {"bit_size",
        {{"i", SameIntOrUnsigned, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
````
- **L385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bessel_j1", {{"x", SameReal}}, SameReal},`.
  **L385 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bessel_j1", {{"x", SameReal}}, SameReal},`。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bessel_jn", {{"n", AnyInt}, {"x", SameReal}}, SameReal},`.
  **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bessel_jn", {{"n", AnyInt}, {"x", SameReal}}, SameReal},`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bessel_jn",`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bessel_jn",`。
- **L388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"n1", AnyInt, Rank::scalar}, {"n2", AnyInt, Rank::scalar},`.
  **L388 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"n1", AnyInt, Rank::scalar}, {"n2", AnyInt, Rank::scalar},`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"x", SameReal, Rank::scalar}},`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"x", SameReal, Rank::scalar}},`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameReal, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameReal, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bessel_y0", {{"x", SameReal}}, SameReal},`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bessel_y0", {{"x", SameReal}}, SameReal},`。
- **L392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bessel_y1", {{"x", SameReal}}, SameReal},`.
  **L392 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bessel_y1", {{"x", SameReal}}, SameReal},`。
- **L393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bessel_yn", {{"n", AnyInt}, {"x", SameReal}}, SameReal},`.
  **L393 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bessel_yn", {{"n", AnyInt}, {"x", SameReal}}, SameReal},`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bessel_yn",`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bessel_yn",`。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"n1", AnyInt, Rank::scalar}, {"n2", AnyInt, Rank::scalar},`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"n1", AnyInt, Rank::scalar}, {"n2", AnyInt, Rank::scalar},`。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"x", SameReal, Rank::scalar}},`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"x", SameReal, Rank::scalar}},`。
- **L397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameReal, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L397 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameReal, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bge",`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bge",`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},`。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},`.
  **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},`。
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bgt",`.
  **L402 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bgt",`。
- **L403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},`.
  **L403 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},`。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},`。
- **L405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L405 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"bit_size",`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"bit_size",`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", SameIntOrUnsigned, Rank::anyOrAssumedRank, Optionality::required,`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", SameIntOrUnsigned, Rank::anyOrAssumedRank, Optionality::required,`。
- **L408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L408 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。

### Lines 409-432

````cpp
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        SameInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"ble",
        {{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},
            {"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},
        DefaultLogical},
    {"blt",
        {{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},
            {"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},
        DefaultLogical},
    {"btest", {{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ}, {"pos", AnyInt}},
        DefaultLogical},
    {"ceiling", {{"a", AnyReal}, DefaultingKIND}, KINDInt},
    {"char", {{"i", AnyInt, Rank::elementalOrBOZ}, DefaultingKIND}, KINDChar},
    {"chdir", {{"name", DefaultChar, Rank::scalar, Optionality::required}},
        DefaultInt},
    {"cmplx", {{"x", AnyComplex}, DefaultingKIND}, KINDComplex},
    {"cmplx",
        {{"x", AnyIntUnsignedOrReal, Rank::elementalOrBOZ},
            {"y", AnyIntUnsignedOrReal, Rank::elementalOrBOZ,
                Optionality::optional},
            DefaultingKIND},
        KINDComplex},
    {"command_argument_count", {}, DefaultInt, Rank::scalar,
````
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ble",`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ble",`。
- **L412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},`.
  **L412 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},`。
- **L413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},`.
  **L413 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"blt",`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"blt",`。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},`.
  **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ},`。
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},`.
  **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"j", AnyIntOrUnsigned, Rank::elementalOrBOZ}},`。
- **L418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L418 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"btest", {{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ}, {"pos", AnyInt}},`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"btest", {{"i", AnyIntOrUnsigned, Rank::elementalOrBOZ}, {"pos", AnyInt}},`。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ceiling", {{"a", AnyReal}, DefaultingKIND}, KINDInt},`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ceiling", {{"a", AnyReal}, DefaultingKIND}, KINDInt},`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"char", {{"i", AnyInt, Rank::elementalOrBOZ}, DefaultingKIND}, KINDChar},`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"char", {{"i", AnyInt, Rank::elementalOrBOZ}, DefaultingKIND}, KINDChar},`。
- **L423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"chdir", {{"name", DefaultChar, Rank::scalar, Optionality::required}},`.
  **L423 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"chdir", {{"name", DefaultChar, Rank::scalar, Optionality::required}},`。
- **L424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt},`.
  **L424 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt},`。
- **L425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cmplx", {{"x", AnyComplex}, DefaultingKIND}, KINDComplex},`.
  **L425 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cmplx", {{"x", AnyComplex}, DefaultingKIND}, KINDComplex},`。
- **L426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cmplx",`.
  **L426 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cmplx",`。
- **L427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyIntUnsignedOrReal, Rank::elementalOrBOZ},`.
  **L427 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyIntUnsignedOrReal, Rank::elementalOrBOZ},`。
- **L428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"y", AnyIntUnsignedOrReal, Rank::elementalOrBOZ,`.
  **L428 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"y", AnyIntUnsignedOrReal, Rank::elementalOrBOZ,`。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::optional},`.
  **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::optional},`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultingKIND},`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultingKIND},`。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDComplex},`.
  **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDComplex},`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"command_argument_count", {}, DefaultInt, Rank::scalar,`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"command_argument_count", {}, DefaultInt, Rank::scalar,`。

### Lines 433-456

````cpp
        IntrinsicClass::transformationalFunction},
    {"conjg", {{"z", SameComplex}}, SameComplex},
    {"cos", {{"x", SameFloating}}, SameFloating},
    {"cosd", {{"x", SameFloating}}, SameFloating},
    {"cospi", {{"x", SameFloating}}, SameFloating},
    {"cosh", {{"x", SameFloating}}, SameFloating},
    {"coshape", {{"coarray", AnyData, Rank::coarray}, SizeDefaultKIND}, KINDInt,
        Rank::vector, IntrinsicClass::inquiryFunction},
    {"count", {{"mask", AnyLogical, Rank::array}, OptionalDIM, DefaultingKIND},
        KINDInt, Rank::dimReduced, IntrinsicClass::transformationalFunction},
    {"cshift",
        {{"array", SameType, Rank::array},
            {"shift", AnyInt, Rank::dimRemovedOrScalar}, OptionalDIM},
        SameType, Rank::conformable, IntrinsicClass::transformationalFunction},
    {"dble", {{"a", AnyNumeric, Rank::elementalOrBOZ}}, DoublePrecision},
    {"digits",
        {{"x", AnyIntUnsignedOrReal, Rank::anyOrAssumedRank,
            Optionality::required, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"dim", {{"x", OperandIntOrReal}, {"y", OperandIntOrReal}},
        OperandIntOrReal},
    {"dot_product",
        {{"vector_a", AnyLogical, Rank::vector},
````
- **L433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L433 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"conjg", {{"z", SameComplex}}, SameComplex},`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"conjg", {{"z", SameComplex}}, SameComplex},`。
- **L435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cos", {{"x", SameFloating}}, SameFloating},`.
  **L435 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cos", {{"x", SameFloating}}, SameFloating},`。
- **L436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cosd", {{"x", SameFloating}}, SameFloating},`.
  **L436 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cosd", {{"x", SameFloating}}, SameFloating},`。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cospi", {{"x", SameFloating}}, SameFloating},`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cospi", {{"x", SameFloating}}, SameFloating},`。
- **L438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cosh", {{"x", SameFloating}}, SameFloating},`.
  **L438 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cosh", {{"x", SameFloating}}, SameFloating},`。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"coshape", {{"coarray", AnyData, Rank::coarray}, SizeDefaultKIND}, KINDInt,`.
  **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"coshape", {{"coarray", AnyData, Rank::coarray}, SizeDefaultKIND}, KINDInt,`。
- **L440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::vector, IntrinsicClass::inquiryFunction},`.
  **L440 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::vector, IntrinsicClass::inquiryFunction},`。
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"count", {{"mask", AnyLogical, Rank::array}, OptionalDIM, DefaultingKIND},`.
  **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"count", {{"mask", AnyLogical, Rank::array}, OptionalDIM, DefaultingKIND},`。
- **L442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::dimReduced, IntrinsicClass::transformationalFunction},`.
  **L442 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::dimReduced, IntrinsicClass::transformationalFunction},`。
- **L443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cshift",`.
  **L443 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cshift",`。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameType, Rank::array},`.
  **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameType, Rank::array},`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"shift", AnyInt, Rank::dimRemovedOrScalar}, OptionalDIM},`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"shift", AnyInt, Rank::dimRemovedOrScalar}, OptionalDIM},`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::conformable, IntrinsicClass::transformationalFunction},`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::conformable, IntrinsicClass::transformationalFunction},`。
- **L447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dble", {{"a", AnyNumeric, Rank::elementalOrBOZ}}, DoublePrecision},`.
  **L447 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dble", {{"a", AnyNumeric, Rank::elementalOrBOZ}}, DoublePrecision},`。
- **L448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"digits",`.
  **L448 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"digits",`。
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyIntUnsignedOrReal, Rank::anyOrAssumedRank,`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyIntUnsignedOrReal, Rank::anyOrAssumedRank,`。
- **L450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::required, common::Intent::In,`.
  **L450 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::required, common::Intent::In,`。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dim", {{"x", OperandIntOrReal}, {"y", OperandIntOrReal}},`.
  **L453 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dim", {{"x", OperandIntOrReal}, {"y", OperandIntOrReal}},`。
- **L454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandIntOrReal},`.
  **L454 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandIntOrReal},`。
- **L455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dot_product",`.
  **L455 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dot_product",`。
- **L456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"vector_a", AnyLogical, Rank::vector},`.
  **L456 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"vector_a", AnyLogical, Rank::vector},`。

### Lines 457-480

````cpp
            {"vector_b", AnyLogical, Rank::vector}},
        ResultLogical, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"dot_product",
        {{"vector_a", AnyComplex, Rank::vector},
            {"vector_b", AnyNumeric, Rank::vector}},
        ResultNumeric, Rank::scalar, // conjugates vector_a
        IntrinsicClass::transformationalFunction},
    {"dot_product",
        {{"vector_a", AnyIntUnsignedOrReal, Rank::vector},
            {"vector_b", AnyNumeric, Rank::vector}},
        ResultNumeric, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"dprod", {{"x", DefaultReal}, {"y", DefaultReal}}, DoublePrecision},
    {"dsecnds",
        {{"refTime", TypePattern{RealType, KindCode::exactKind, 8},
            Rank::scalar}},
        TypePattern{RealType, KindCode::exactKind, 8}, Rank::scalar},
    {"dshiftl",
        {{"i", SameIntOrUnsigned},
            {"j", SameIntOrUnsigned, Rank::elementalOrBOZ}, {"shift", AnyInt}},
        SameIntOrUnsigned},
    {"dshiftl", {{"i", BOZ}, {"j", SameIntOrUnsigned}, {"shift", AnyInt}},
        SameIntOrUnsigned},
    {"dshiftr",
        {{"i", SameIntOrUnsigned},
````
- **L457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"vector_b", AnyLogical, Rank::vector}},`.
  **L457 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"vector_b", AnyLogical, Rank::vector}},`。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ResultLogical, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`ResultLogical, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dot_product",`.
  **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dot_product",`。
- **L460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"vector_a", AnyComplex, Rank::vector},`.
  **L460 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"vector_a", AnyComplex, Rank::vector},`。
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"vector_b", AnyNumeric, Rank::vector}},`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"vector_b", AnyNumeric, Rank::vector}},`。
- **L462 EN**: Continues the surrounding expression or declaration: `ResultNumeric, Rank::scalar, // conjugates vector_a`.
  **L462 CN**: 继续构造周围的表达式或声明：`ResultNumeric, Rank::scalar, // conjugates vector_a`。
- **L463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L463 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dot_product",`.
  **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dot_product",`。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"vector_a", AnyIntUnsignedOrReal, Rank::vector},`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"vector_a", AnyIntUnsignedOrReal, Rank::vector},`。
- **L466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"vector_b", AnyNumeric, Rank::vector}},`.
  **L466 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"vector_b", AnyNumeric, Rank::vector}},`。
- **L467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ResultNumeric, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L467 CN**: 继续一个多行参数列表、初始化器或聚合项：`ResultNumeric, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dprod", {{"x", DefaultReal}, {"y", DefaultReal}}, DoublePrecision},`.
  **L468 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dprod", {{"x", DefaultReal}, {"y", DefaultReal}}, DoublePrecision},`。
- **L469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dsecnds",`.
  **L469 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dsecnds",`。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"refTime", TypePattern{RealType, KindCode::exactKind, 8},`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"refTime", TypePattern{RealType, KindCode::exactKind, 8},`。
- **L471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar}},`.
  **L471 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar}},`。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{RealType, KindCode::exactKind, 8}, Rank::scalar},`.
  **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{RealType, KindCode::exactKind, 8}, Rank::scalar},`。
- **L473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dshiftl",`.
  **L473 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dshiftl",`。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", SameIntOrUnsigned},`.
  **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", SameIntOrUnsigned},`。
- **L475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"j", SameIntOrUnsigned, Rank::elementalOrBOZ}, {"shift", AnyInt}},`.
  **L475 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"j", SameIntOrUnsigned, Rank::elementalOrBOZ}, {"shift", AnyInt}},`。
- **L476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L476 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dshiftl", {{"i", BOZ}, {"j", SameIntOrUnsigned}, {"shift", AnyInt}},`.
  **L477 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dshiftl", {{"i", BOZ}, {"j", SameIntOrUnsigned}, {"shift", AnyInt}},`。
- **L478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L478 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dshiftr",`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dshiftr",`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", SameIntOrUnsigned},`.
  **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", SameIntOrUnsigned},`。

### Lines 481-504

````cpp
            {"j", SameIntOrUnsigned, Rank::elementalOrBOZ}, {"shift", AnyInt}},
        SameIntOrUnsigned},
    {"dshiftr", {{"i", BOZ}, {"j", SameIntOrUnsigned}, {"shift", AnyInt}},
        SameIntOrUnsigned},
    {"eoshift",
        {{"array", SameType, Rank::array},
            {"shift", AnyInt, Rank::dimRemovedOrScalar},
            // BOUNDARY= is not optional for non-intrinsic types
            {"boundary", SameType, Rank::dimRemovedOrScalar}, OptionalDIM},
        SameType, Rank::conformable, IntrinsicClass::transformationalFunction},
    {"eoshift",
        {{"array", SameIntrinsic, Rank::array},
            {"shift", AnyInt, Rank::dimRemovedOrScalar},
            {"boundary", SameIntrinsic, Rank::dimRemovedOrScalar,
                Optionality::optional},
            OptionalDIM},
        SameIntrinsic, Rank::conformable,
        IntrinsicClass::transformationalFunction},
    {"epsilon",
        {{"x", SameReal, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        SameReal, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"erf", {{"x", SameReal}}, SameReal},
````
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"j", SameIntOrUnsigned, Rank::elementalOrBOZ}, {"shift", AnyInt}},`.
  **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"j", SameIntOrUnsigned, Rank::elementalOrBOZ}, {"shift", AnyInt}},`。
- **L482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L482 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"dshiftr", {{"i", BOZ}, {"j", SameIntOrUnsigned}, {"shift", AnyInt}},`.
  **L483 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"dshiftr", {{"i", BOZ}, {"j", SameIntOrUnsigned}, {"shift", AnyInt}},`。
- **L484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L484 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"eoshift",`.
  **L485 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"eoshift",`。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameType, Rank::array},`.
  **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameType, Rank::array},`。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"shift", AnyInt, Rank::dimRemovedOrScalar},`.
  **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"shift", AnyInt, Rank::dimRemovedOrScalar},`。
- **L488 EN**: Comment explains nearby logic, intent, or metadata: `BOUNDARY= is not optional for non-intrinsic types`.
  **L488 CN**: 注释说明附近代码的逻辑、意图或元数据：`BOUNDARY= is not optional for non-intrinsic types`。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"boundary", SameType, Rank::dimRemovedOrScalar}, OptionalDIM},`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"boundary", SameType, Rank::dimRemovedOrScalar}, OptionalDIM},`。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::conformable, IntrinsicClass::transformationalFunction},`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::conformable, IntrinsicClass::transformationalFunction},`。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"eoshift",`.
  **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"eoshift",`。
- **L492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameIntrinsic, Rank::array},`.
  **L492 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameIntrinsic, Rank::array},`。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"shift", AnyInt, Rank::dimRemovedOrScalar},`.
  **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"shift", AnyInt, Rank::dimRemovedOrScalar},`。
- **L494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"boundary", SameIntrinsic, Rank::dimRemovedOrScalar,`.
  **L494 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"boundary", SameIntrinsic, Rank::dimRemovedOrScalar,`。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::optional},`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::optional},`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OptionalDIM},`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`OptionalDIM},`。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntrinsic, Rank::conformable,`.
  **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntrinsic, Rank::conformable,`。
- **L498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L498 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"epsilon",`.
  **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"epsilon",`。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", SameReal, Rank::anyOrAssumedRank, Optionality::required,`.
  **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", SameReal, Rank::anyOrAssumedRank, Optionality::required,`。
- **L501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L501 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameReal, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L503 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameReal, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"erf", {{"x", SameReal}}, SameReal},`.
  **L504 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"erf", {{"x", SameReal}}, SameReal},`。

### Lines 505-528

````cpp
    {"erfc", {{"x", SameReal}}, SameReal},
    {"erfc_scaled", {{"x", SameReal}}, SameReal},
    {"etime",
        {{"values", TypePattern{RealType, KindCode::exactKind, 4}, Rank::vector,
            Optionality::required, common::Intent::Out}},
        TypePattern{RealType, KindCode::exactKind, 4}},
    {"exp", {{"x", SameFloating}}, SameFloating},
    {"exp", {{"x", SameFloating}}, SameFloating},
    {"exponent", {{"x", AnyReal}}, DefaultInt},
    {"exp", {{"x", SameFloating}}, SameFloating},
    {"extends_type_of",
        {{"a", ExtensibleDerived, Rank::anyOrAssumedRank, Optionality::required,
             common::Intent::In, {ArgFlag::canBeMoldNull}},
            {"mold", ExtensibleDerived, Rank::anyOrAssumedRank,
                Optionality::required, common::Intent::In,
                {ArgFlag::canBeMoldNull}}},
        DefaultLogical, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"__builtin_f_c_string",
        {{"string", CChar, Rank::scalar},
            {"asis", AnyLogical, Rank::scalar, Optionality::optional}},
        CChar, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"failed_images", {OptionalTEAM, SizeDefaultKIND}, KINDInt, Rank::vector,
        IntrinsicClass::transformationalFunction},
    {"findloc",
````
- **L505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"erfc", {{"x", SameReal}}, SameReal},`.
  **L505 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"erfc", {{"x", SameReal}}, SameReal},`。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"erfc_scaled", {{"x", SameReal}}, SameReal},`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"erfc_scaled", {{"x", SameReal}}, SameReal},`。
- **L507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"etime",`.
  **L507 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"etime",`。
- **L508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"values", TypePattern{RealType, KindCode::exactKind, 4}, Rank::vector,`.
  **L508 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"values", TypePattern{RealType, KindCode::exactKind, 4}, Rank::vector,`。
- **L509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::required, common::Intent::Out}},`.
  **L509 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::required, common::Intent::Out}},`。
- **L510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{RealType, KindCode::exactKind, 4}},`.
  **L510 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{RealType, KindCode::exactKind, 4}},`。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"exp", {{"x", SameFloating}}, SameFloating},`.
  **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"exp", {{"x", SameFloating}}, SameFloating},`。
- **L512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"exp", {{"x", SameFloating}}, SameFloating},`.
  **L512 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"exp", {{"x", SameFloating}}, SameFloating},`。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"exponent", {{"x", AnyReal}}, DefaultInt},`.
  **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"exponent", {{"x", AnyReal}}, DefaultInt},`。
- **L514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"exp", {{"x", SameFloating}}, SameFloating},`.
  **L514 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"exp", {{"x", SameFloating}}, SameFloating},`。
- **L515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"extends_type_of",`.
  **L515 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"extends_type_of",`。
- **L516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", ExtensibleDerived, Rank::anyOrAssumedRank, Optionality::required,`.
  **L516 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", ExtensibleDerived, Rank::anyOrAssumedRank, Optionality::required,`。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In, {ArgFlag::canBeMoldNull}},`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In, {ArgFlag::canBeMoldNull}},`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mold", ExtensibleDerived, Rank::anyOrAssumedRank,`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mold", ExtensibleDerived, Rank::anyOrAssumedRank,`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::required, common::Intent::In,`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::required, common::Intent::In,`。
- **L520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull}}},`.
  **L520 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull}}},`。
- **L521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L521 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_f_c_string",`.
  **L522 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_f_c_string",`。
- **L523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"string", CChar, Rank::scalar},`.
  **L523 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"string", CChar, Rank::scalar},`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"asis", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"asis", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CChar, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`CChar, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"failed_images", {OptionalTEAM, SizeDefaultKIND}, KINDInt, Rank::vector,`.
  **L526 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"failed_images", {OptionalTEAM, SizeDefaultKIND}, KINDInt, Rank::vector,`。
- **L527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L527 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"findloc",`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"findloc",`。

### Lines 529-552

````cpp
        {{"array", AnyNumeric, Rank::array},
            {"value", AnyNumeric, Rank::scalar}, RequiredDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
        KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},
    {"findloc",
        {{"array", AnyNumeric, Rank::array},
            {"value", AnyNumeric, Rank::scalar}, MissingDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
        KINDInt, Rank::vector, IntrinsicClass::transformationalFunction},
    {"findloc",
        {{"array", SameCharNoLen, Rank::array},
            {"value", SameCharNoLen, Rank::scalar}, RequiredDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
        KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},
    {"findloc",
        {{"array", SameCharNoLen, Rank::array},
            {"value", SameCharNoLen, Rank::scalar}, MissingDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
        KINDInt, Rank::vector, IntrinsicClass::transformationalFunction},
    {"findloc",
````
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyNumeric, Rank::array},`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyNumeric, Rank::array},`。
- **L530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyNumeric, Rank::scalar}, RequiredDIM, OptionalMASK,`.
  **L530 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyNumeric, Rank::scalar}, RequiredDIM, OptionalMASK,`。
- **L531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L531 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L532 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`.
  **L533 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"findloc",`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"findloc",`。
- **L535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyNumeric, Rank::array},`.
  **L535 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyNumeric, Rank::array},`。
- **L536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyNumeric, Rank::scalar}, MissingDIM, OptionalMASK,`.
  **L536 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyNumeric, Rank::scalar}, MissingDIM, OptionalMASK,`。
- **L537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L537 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"findloc",`.
  **L540 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"findloc",`。
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameCharNoLen, Rank::array},`.
  **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameCharNoLen, Rank::array},`。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", SameCharNoLen, Rank::scalar}, RequiredDIM, OptionalMASK,`.
  **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", SameCharNoLen, Rank::scalar}, RequiredDIM, OptionalMASK,`。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`.
  **L545 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`。
- **L546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"findloc",`.
  **L546 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"findloc",`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameCharNoLen, Rank::array},`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameCharNoLen, Rank::array},`。
- **L548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", SameCharNoLen, Rank::scalar}, MissingDIM, OptionalMASK,`.
  **L548 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", SameCharNoLen, Rank::scalar}, MissingDIM, OptionalMASK,`。
- **L549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L549 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"findloc",`.
  **L552 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"findloc",`。

### Lines 553-576

````cpp
        {{"array", AnyLogical, Rank::array},
            {"value", AnyLogical, Rank::scalar}, RequiredDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
        KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},
    {"findloc",
        {{"array", AnyLogical, Rank::array},
            {"value", AnyLogical, Rank::scalar}, MissingDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
        KINDInt, Rank::vector, IntrinsicClass::transformationalFunction},
    {"floor", {{"a", AnyReal}, DefaultingKIND}, KINDInt},
    {"fraction", {{"x", SameReal}}, SameReal},
    {"fseek",
        {{"unit", AnyInt, Rank::scalar}, {"offset", AnyInt, Rank::scalar},
            {"whence", AnyInt, Rank::scalar}},
        DefaultInt, Rank::scalar},
    {"ftell", {{"unit", AnyInt, Rank::scalar}},
        TypePattern{IntType, KindCode::exactKind, 8}, Rank::scalar},
    {"gamma", {{"x", SameReal}}, SameReal},
    {"get_team", {{"level", DefaultInt, Rank::scalar, Optionality::optional}},
        TeamType, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"getcwd",
        {{"c", DefaultChar, Rank::scalar, Optionality::required,
````
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyLogical, Rank::array},`.
  **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyLogical, Rank::array},`。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyLogical, Rank::scalar}, RequiredDIM, OptionalMASK,`.
  **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyLogical, Rank::scalar}, RequiredDIM, OptionalMASK,`。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L556 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`.
  **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"findloc",`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"findloc",`。
- **L559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyLogical, Rank::array},`.
  **L559 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyLogical, Rank::array},`。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyLogical, Rank::scalar}, MissingDIM, OptionalMASK,`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyLogical, Rank::scalar}, MissingDIM, OptionalMASK,`。
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L562 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L563 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"floor", {{"a", AnyReal}, DefaultingKIND}, KINDInt},`.
  **L564 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"floor", {{"a", AnyReal}, DefaultingKIND}, KINDInt},`。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"fraction", {{"x", SameReal}}, SameReal},`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"fraction", {{"x", SameReal}}, SameReal},`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"fseek",`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"fseek",`。
- **L567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"unit", AnyInt, Rank::scalar}, {"offset", AnyInt, Rank::scalar},`.
  **L567 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"unit", AnyInt, Rank::scalar}, {"offset", AnyInt, Rank::scalar},`。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"whence", AnyInt, Rank::scalar}},`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"whence", AnyInt, Rank::scalar}},`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar},`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar},`。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ftell", {{"unit", AnyInt, Rank::scalar}},`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ftell", {{"unit", AnyInt, Rank::scalar}},`。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 8}, Rank::scalar},`.
  **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 8}, Rank::scalar},`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"gamma", {{"x", SameReal}}, SameReal},`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"gamma", {{"x", SameReal}}, SameReal},`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"get_team", {{"level", DefaultInt, Rank::scalar, Optionality::optional}},`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"get_team", {{"level", DefaultInt, Rank::scalar, Optionality::optional}},`。
- **L574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TeamType, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L574 CN**: 继续一个多行参数列表、初始化器或聚合项：`TeamType, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"getcwd",`.
  **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"getcwd",`。
- **L576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"c", DefaultChar, Rank::scalar, Optionality::required,`.
  **L576 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"c", DefaultChar, Rank::scalar, Optionality::required,`。

### Lines 577-600

````cpp
            common::Intent::Out}},
        TypePattern{IntType, KindCode::greaterOrEqualToKind, 4}},
    {"getgid", {}, DefaultInt},
    {"getpid", {}, DefaultInt},
    {"getuid", {}, DefaultInt},
    {"hostnm",
        {{"c", DefaultChar, Rank::scalar, Optionality::required,
            common::Intent::Out}},
        TypePattern{IntType, KindCode::greaterOrEqualToKind, 4}},
    {"huge",
        {{"x", SameIntUnsignedOrReal, Rank::anyOrAssumedRank,
            Optionality::required, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        SameIntUnsignedOrReal, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"hypot", {{"x", OperandReal}, {"y", OperandReal}}, OperandReal},
    {"iachar", {{"c", AnyChar}, DefaultingKIND}, KINDInt},
    {"iall",
        {{"array", SameIntOrUnsigned, Rank::array}, RequiredDIM, OptionalMASK},
        SameIntOrUnsigned, Rank::dimReduced,
        IntrinsicClass::transformationalFunction},
    {"iall",
        {{"array", SameIntOrUnsigned, Rank::array}, MissingDIM, OptionalMASK},
        SameIntOrUnsigned, Rank::scalar,
        IntrinsicClass::transformationalFunction},
````
- **L577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L577 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::greaterOrEqualToKind, 4}},`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::greaterOrEqualToKind, 4}},`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"getgid", {}, DefaultInt},`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"getgid", {}, DefaultInt},`。
- **L580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"getpid", {}, DefaultInt},`.
  **L580 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"getpid", {}, DefaultInt},`。
- **L581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"getuid", {}, DefaultInt},`.
  **L581 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"getuid", {}, DefaultInt},`。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"hostnm",`.
  **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"hostnm",`。
- **L583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"c", DefaultChar, Rank::scalar, Optionality::required,`.
  **L583 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"c", DefaultChar, Rank::scalar, Optionality::required,`。
- **L584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L584 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::greaterOrEqualToKind, 4}},`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::greaterOrEqualToKind, 4}},`。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"huge",`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"huge",`。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", SameIntUnsignedOrReal, Rank::anyOrAssumedRank,`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", SameIntUnsignedOrReal, Rank::anyOrAssumedRank,`。
- **L588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::required, common::Intent::In,`.
  **L588 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::required, common::Intent::In,`。
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntUnsignedOrReal, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntUnsignedOrReal, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"hypot", {{"x", OperandReal}, {"y", OperandReal}}, OperandReal},`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"hypot", {{"x", OperandReal}, {"y", OperandReal}}, OperandReal},`。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iachar", {{"c", AnyChar}, DefaultingKIND}, KINDInt},`.
  **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iachar", {{"c", AnyChar}, DefaultingKIND}, KINDInt},`。
- **L593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iall",`.
  **L593 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iall",`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameIntOrUnsigned, Rank::array}, RequiredDIM, OptionalMASK},`.
  **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameIntOrUnsigned, Rank::array}, RequiredDIM, OptionalMASK},`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned, Rank::dimReduced,`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned, Rank::dimReduced,`。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iall",`.
  **L597 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iall",`。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameIntOrUnsigned, Rank::array}, MissingDIM, OptionalMASK},`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameIntOrUnsigned, Rank::array}, MissingDIM, OptionalMASK},`。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned, Rank::scalar,`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned, Rank::scalar,`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。

### Lines 601-624

````cpp
    {"iany",
        {{"array", SameIntOrUnsigned, Rank::array}, RequiredDIM, OptionalMASK},
        SameIntOrUnsigned, Rank::dimReduced,
        IntrinsicClass::transformationalFunction},
    {"iany",
        {{"array", SameIntOrUnsigned, Rank::array}, MissingDIM, OptionalMASK},
        SameIntOrUnsigned, Rank::scalar,
        IntrinsicClass::transformationalFunction},
    {"iparity",
        {{"array", SameIntOrUnsigned, Rank::array}, RequiredDIM, OptionalMASK},
        SameIntOrUnsigned, Rank::dimReduced,
        IntrinsicClass::transformationalFunction},
    {"iparity",
        {{"array", SameIntOrUnsigned, Rank::array}, MissingDIM, OptionalMASK},
        SameIntOrUnsigned, Rank::scalar,
        IntrinsicClass::transformationalFunction},
    {"iand", {{"i", OperandInt}, {"j", OperandInt, Rank::elementalOrBOZ}},
        OperandInt},
    {"iand",
        {{"i", OperandUnsigned}, {"j", OperandUnsigned, Rank::elementalOrBOZ}},
        OperandUnsigned},
    {"iand", {{"i", BOZ}, {"j", SameIntOrUnsigned}}, SameIntOrUnsigned},
    {"ibclr", {{"i", SameIntOrUnsigned}, {"pos", AnyInt}}, SameIntOrUnsigned},
    {"ibits", {{"i", SameIntOrUnsigned}, {"pos", AnyInt}, {"len", AnyInt}},
````
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iany",`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iany",`。
- **L602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameIntOrUnsigned, Rank::array}, RequiredDIM, OptionalMASK},`.
  **L602 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameIntOrUnsigned, Rank::array}, RequiredDIM, OptionalMASK},`。
- **L603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned, Rank::dimReduced,`.
  **L603 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned, Rank::dimReduced,`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iany",`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iany",`。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameIntOrUnsigned, Rank::array}, MissingDIM, OptionalMASK},`.
  **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameIntOrUnsigned, Rank::array}, MissingDIM, OptionalMASK},`。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned, Rank::scalar,`.
  **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned, Rank::scalar,`。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iparity",`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iparity",`。
- **L610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameIntOrUnsigned, Rank::array}, RequiredDIM, OptionalMASK},`.
  **L610 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameIntOrUnsigned, Rank::array}, RequiredDIM, OptionalMASK},`。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned, Rank::dimReduced,`.
  **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned, Rank::dimReduced,`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iparity",`.
  **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iparity",`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameIntOrUnsigned, Rank::array}, MissingDIM, OptionalMASK},`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameIntOrUnsigned, Rank::array}, MissingDIM, OptionalMASK},`。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned, Rank::scalar,`.
  **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned, Rank::scalar,`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iand", {{"i", OperandInt}, {"j", OperandInt, Rank::elementalOrBOZ}},`.
  **L617 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iand", {{"i", OperandInt}, {"j", OperandInt, Rank::elementalOrBOZ}},`。
- **L618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandInt},`.
  **L618 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandInt},`。
- **L619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iand",`.
  **L619 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iand",`。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", OperandUnsigned}, {"j", OperandUnsigned, Rank::elementalOrBOZ}},`.
  **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", OperandUnsigned}, {"j", OperandUnsigned, Rank::elementalOrBOZ}},`。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandUnsigned},`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandUnsigned},`。
- **L622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"iand", {{"i", BOZ}, {"j", SameIntOrUnsigned}}, SameIntOrUnsigned},`.
  **L622 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"iand", {{"i", BOZ}, {"j", SameIntOrUnsigned}}, SameIntOrUnsigned},`。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ibclr", {{"i", SameIntOrUnsigned}, {"pos", AnyInt}}, SameIntOrUnsigned},`.
  **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ibclr", {{"i", SameIntOrUnsigned}, {"pos", AnyInt}}, SameIntOrUnsigned},`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ibits", {{"i", SameIntOrUnsigned}, {"pos", AnyInt}, {"len", AnyInt}},`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ibits", {{"i", SameIntOrUnsigned}, {"pos", AnyInt}, {"len", AnyInt}},`。

### Lines 625-648

````cpp
        SameIntOrUnsigned},
    {"ibset", {{"i", SameIntOrUnsigned}, {"pos", AnyInt}}, SameIntOrUnsigned},
    {"ichar", {{"c", AnyChar}, DefaultingKIND}, KINDInt},
    {"ieor", {{"i", OperandInt}, {"j", OperandInt, Rank::elementalOrBOZ}},
        OperandInt},
    {"ieor",
        {{"i", OperandUnsigned}, {"j", OperandUnsigned, Rank::elementalOrBOZ}},
        OperandUnsigned},
    {"ieor", {{"i", BOZ}, {"j", SameIntOrUnsigned}}, SameIntOrUnsigned},
    {"image_index",
        {{"coarray", AnyData, Rank::coarray}, {"sub", AnyInt, Rank::vector}},
        DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"image_index",
        {{"coarray", AnyData, Rank::coarray}, {"sub", AnyInt, Rank::vector},
            {"team", TeamType, Rank::scalar}},
        DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"image_index",
        {{"coarray", AnyData, Rank::coarray}, {"sub", AnyInt, Rank::vector},
            {"team_number", AnyInt, Rank::scalar}},
        DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"image_status", {{"image", SameInt}, OptionalTEAM}, DefaultInt},
    {"index",
        {{"string", SameCharNoLen}, {"substring", SameCharNoLen},
            {"back", AnyLogical, Rank::elemental, Optionality::optional},
````
- **L625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L625 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ibset", {{"i", SameIntOrUnsigned}, {"pos", AnyInt}}, SameIntOrUnsigned},`.
  **L626 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ibset", {{"i", SameIntOrUnsigned}, {"pos", AnyInt}}, SameIntOrUnsigned},`。
- **L627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ichar", {{"c", AnyChar}, DefaultingKIND}, KINDInt},`.
  **L627 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ichar", {{"c", AnyChar}, DefaultingKIND}, KINDInt},`。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ieor", {{"i", OperandInt}, {"j", OperandInt, Rank::elementalOrBOZ}},`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ieor", {{"i", OperandInt}, {"j", OperandInt, Rank::elementalOrBOZ}},`。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandInt},`.
  **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandInt},`。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ieor",`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ieor",`。
- **L631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", OperandUnsigned}, {"j", OperandUnsigned, Rank::elementalOrBOZ}},`.
  **L631 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", OperandUnsigned}, {"j", OperandUnsigned, Rank::elementalOrBOZ}},`。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandUnsigned},`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandUnsigned},`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ieor", {{"i", BOZ}, {"j", SameIntOrUnsigned}}, SameIntOrUnsigned},`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ieor", {{"i", BOZ}, {"j", SameIntOrUnsigned}}, SameIntOrUnsigned},`。
- **L634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"image_index",`.
  **L634 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"image_index",`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"coarray", AnyData, Rank::coarray}, {"sub", AnyInt, Rank::vector}},`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"coarray", AnyData, Rank::coarray}, {"sub", AnyInt, Rank::vector}},`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"image_index",`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"image_index",`。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"coarray", AnyData, Rank::coarray}, {"sub", AnyInt, Rank::vector},`.
  **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"coarray", AnyData, Rank::coarray}, {"sub", AnyInt, Rank::vector},`。
- **L639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"team", TeamType, Rank::scalar}},`.
  **L639 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"team", TeamType, Rank::scalar}},`。
- **L640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L640 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"image_index",`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"image_index",`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"coarray", AnyData, Rank::coarray}, {"sub", AnyInt, Rank::vector},`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"coarray", AnyData, Rank::coarray}, {"sub", AnyInt, Rank::vector},`。
- **L643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"team_number", AnyInt, Rank::scalar}},`.
  **L643 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"team_number", AnyInt, Rank::scalar}},`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"image_status", {{"image", SameInt}, OptionalTEAM}, DefaultInt},`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"image_status", {{"image", SameInt}, OptionalTEAM}, DefaultInt},`。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"index",`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"index",`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"string", SameCharNoLen}, {"substring", SameCharNoLen},`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"string", SameCharNoLen}, {"substring", SameCharNoLen},`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::elemental, Optionality::optional},`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::elemental, Optionality::optional},`。

### Lines 649-672

````cpp
            DefaultingKIND},
        KINDInt},
    {"int", {{"a", AnyNumeric, Rank::elementalOrBOZ}, DefaultingKIND}, KINDInt},
    {"int2", {{"a", AnyNumeric, Rank::elementalOrBOZ}},
        TypePattern{IntType, KindCode::exactKind, 2}},
    {"int8", {{"a", AnyNumeric, Rank::elementalOrBOZ}},
        TypePattern{IntType, KindCode::exactKind, 8}},
    {"int_ptr_kind", {}, DefaultInt, Rank::scalar},
    {"ior", {{"i", OperandInt}, {"j", OperandInt, Rank::elementalOrBOZ}},
        OperandInt},
    {"ior",
        {{"i", OperandUnsigned}, {"j", OperandUnsigned, Rank::elementalOrBOZ}},
        OperandUnsigned},
    {"ior", {{"i", BOZ}, {"j", SameIntOrUnsigned}}, SameIntOrUnsigned},
    {"irand",
        {{"i", TypePattern{IntType, KindCode::exactKind, 4}, Rank::scalar,
            Optionality::optional}},
        TypePattern{IntType, KindCode::exactKind, 4}, Rank::scalar,
        IntrinsicClass::impureFunction},
    {"ishft", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}}, SameIntOrUnsigned},
    {"ishftc",
        {{"i", SameIntOrUnsigned}, {"shift", AnyInt},
            {"size", AnyInt, Rank::elemental, Optionality::optional}},
        SameIntOrUnsigned},
````
- **L649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultingKIND},`.
  **L649 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultingKIND},`。
- **L650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt},`.
  **L650 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt},`。
- **L651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"int", {{"a", AnyNumeric, Rank::elementalOrBOZ}, DefaultingKIND}, KINDInt},`.
  **L651 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"int", {{"a", AnyNumeric, Rank::elementalOrBOZ}, DefaultingKIND}, KINDInt},`。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"int2", {{"a", AnyNumeric, Rank::elementalOrBOZ}},`.
  **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"int2", {{"a", AnyNumeric, Rank::elementalOrBOZ}},`。
- **L653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 2}},`.
  **L653 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 2}},`。
- **L654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"int8", {{"a", AnyNumeric, Rank::elementalOrBOZ}},`.
  **L654 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"int8", {{"a", AnyNumeric, Rank::elementalOrBOZ}},`。
- **L655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 8}},`.
  **L655 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 8}},`。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"int_ptr_kind", {}, DefaultInt, Rank::scalar},`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"int_ptr_kind", {}, DefaultInt, Rank::scalar},`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ior", {{"i", OperandInt}, {"j", OperandInt, Rank::elementalOrBOZ}},`.
  **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ior", {{"i", OperandInt}, {"j", OperandInt, Rank::elementalOrBOZ}},`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandInt},`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandInt},`。
- **L659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ior",`.
  **L659 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ior",`。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", OperandUnsigned}, {"j", OperandUnsigned, Rank::elementalOrBOZ}},`.
  **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", OperandUnsigned}, {"j", OperandUnsigned, Rank::elementalOrBOZ}},`。
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandUnsigned},`.
  **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandUnsigned},`。
- **L662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ior", {{"i", BOZ}, {"j", SameIntOrUnsigned}}, SameIntOrUnsigned},`.
  **L662 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ior", {{"i", BOZ}, {"j", SameIntOrUnsigned}}, SameIntOrUnsigned},`。
- **L663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"irand",`.
  **L663 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"irand",`。
- **L664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", TypePattern{IntType, KindCode::exactKind, 4}, Rank::scalar,`.
  **L664 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", TypePattern{IntType, KindCode::exactKind, 4}, Rank::scalar,`。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::optional}},`.
  **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::optional}},`。
- **L666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 4}, Rank::scalar,`.
  **L666 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 4}, Rank::scalar,`。
- **L667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::impureFunction},`.
  **L667 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::impureFunction},`。
- **L668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ishft", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}}, SameIntOrUnsigned},`.
  **L668 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ishft", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}}, SameIntOrUnsigned},`。
- **L669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ishftc",`.
  **L669 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ishftc",`。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", SameIntOrUnsigned}, {"shift", AnyInt},`.
  **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", SameIntOrUnsigned}, {"shift", AnyInt},`。
- **L671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"size", AnyInt, Rank::elemental, Optionality::optional}},`.
  **L671 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"size", AnyInt, Rank::elemental, Optionality::optional}},`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。

### Lines 673-696

````cpp
    {"isnan", {{"a", AnyFloating}}, DefaultLogical},
    {"is_contiguous", {{"array", Addressable, Rank::anyOrAssumedRank}},
        DefaultLogical, Rank::elemental, IntrinsicClass::inquiryFunction},
    {"is_iostat_end", {{"i", AnyInt}}, DefaultLogical},
    {"is_iostat_eor", {{"i", AnyInt}}, DefaultLogical},
    {"izext", {{"i", AnyInt}}, TypePattern{IntType, KindCode::exactKind, 2}},
    {"jzext", {{"i", AnyInt}}, DefaultInt},
    {"kind",
        {{"x", AnyIntrinsic, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultInt, Rank::elemental, IntrinsicClass::inquiryFunction},
    {"lbound",
        {{"array", AnyData, Rank::arrayOrAssumedRank}, RequiredDIM,
            SizeDefaultKIND},
        KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"lbound", {{"array", AnyData, Rank::arrayOrAssumedRank}, SizeDefaultKIND},
        KINDInt, Rank::vector, IntrinsicClass::inquiryFunction},
    {"lcobound",
        {{"coarray", AnyData, Rank::coarray}, OptionalDIM, SizeDefaultKIND},
        KINDInt, Rank::scalarIfDim, IntrinsicClass::inquiryFunction},
    {"leadz", {{"i", AnyInt}}, DefaultInt},
    {"len",
        {{"string", AnyChar, Rank::anyOrAssumedRank, Optionality::required,
````
- **L673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"isnan", {{"a", AnyFloating}}, DefaultLogical},`.
  **L673 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"isnan", {{"a", AnyFloating}}, DefaultLogical},`。
- **L674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"is_contiguous", {{"array", Addressable, Rank::anyOrAssumedRank}},`.
  **L674 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"is_contiguous", {{"array", Addressable, Rank::anyOrAssumedRank}},`。
- **L675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical, Rank::elemental, IntrinsicClass::inquiryFunction},`.
  **L675 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical, Rank::elemental, IntrinsicClass::inquiryFunction},`。
- **L676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"is_iostat_end", {{"i", AnyInt}}, DefaultLogical},`.
  **L676 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"is_iostat_end", {{"i", AnyInt}}, DefaultLogical},`。
- **L677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"is_iostat_eor", {{"i", AnyInt}}, DefaultLogical},`.
  **L677 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"is_iostat_eor", {{"i", AnyInt}}, DefaultLogical},`。
- **L678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"izext", {{"i", AnyInt}}, TypePattern{IntType, KindCode::exactKind, 2}},`.
  **L678 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"izext", {{"i", AnyInt}}, TypePattern{IntType, KindCode::exactKind, 2}},`。
- **L679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"jzext", {{"i", AnyInt}}, DefaultInt},`.
  **L679 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"jzext", {{"i", AnyInt}}, DefaultInt},`。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"kind",`.
  **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"kind",`。
- **L681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyIntrinsic, Rank::anyOrAssumedRank, Optionality::required,`.
  **L681 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyIntrinsic, Rank::anyOrAssumedRank, Optionality::required,`。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L683 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::elemental, IntrinsicClass::inquiryFunction},`.
  **L684 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::elemental, IntrinsicClass::inquiryFunction},`。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"lbound",`.
  **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"lbound",`。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyData, Rank::arrayOrAssumedRank}, RequiredDIM,`.
  **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyData, Rank::arrayOrAssumedRank}, RequiredDIM,`。
- **L687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND},`.
  **L687 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND},`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"lbound", {{"array", AnyData, Rank::arrayOrAssumedRank}, SizeDefaultKIND},`.
  **L689 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"lbound", {{"array", AnyData, Rank::arrayOrAssumedRank}, SizeDefaultKIND},`。
- **L690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::vector, IntrinsicClass::inquiryFunction},`.
  **L690 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::vector, IntrinsicClass::inquiryFunction},`。
- **L691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"lcobound",`.
  **L691 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"lcobound",`。
- **L692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"coarray", AnyData, Rank::coarray}, OptionalDIM, SizeDefaultKIND},`.
  **L692 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"coarray", AnyData, Rank::coarray}, OptionalDIM, SizeDefaultKIND},`。
- **L693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::scalarIfDim, IntrinsicClass::inquiryFunction},`.
  **L693 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::scalarIfDim, IntrinsicClass::inquiryFunction},`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"leadz", {{"i", AnyInt}}, DefaultInt},`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"leadz", {{"i", AnyInt}}, DefaultInt},`。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"len",`.
  **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"len",`。
- **L696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"string", AnyChar, Rank::anyOrAssumedRank, Optionality::required,`.
  **L696 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"string", AnyChar, Rank::anyOrAssumedRank, Optionality::required,`。

### Lines 697-720

````cpp
             common::Intent::In, {ArgFlag::canBeMoldNull}},
            DefaultingKIND},
        KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"len_trim", {{"string", AnyChar}, DefaultingKIND}, KINDInt},
    {"lge", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},
        DefaultLogical},
    {"lgt", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},
        DefaultLogical},
    {"lle", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},
        DefaultLogical},
    {"llt", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},
        DefaultLogical},
    {"lnblnk", {{"string", AnyChar}}, DefaultInt},
    {"loc", {{"x", Addressable, Rank::anyOrAssumedRank}}, SubscriptInt,
        Rank::scalar},
    {"log", {{"x", SameFloating}}, SameFloating},
    {"log10", {{"x", SameReal}}, SameReal},
    {"logical", {{"l", AnyLogical}, DefaultingKIND}, KINDLogical},
    {"log_gamma", {{"x", SameReal}}, SameReal},
    {"malloc", {{"size", AnyInt}}, SubscriptInt},
    {"matmul",
        {{"matrix_a", AnyLogical, Rank::vector},
            {"matrix_b", AnyLogical, Rank::matrix}},
        ResultLogical, Rank::vector, IntrinsicClass::transformationalFunction},
````
- **L697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In, {ArgFlag::canBeMoldNull}},`.
  **L697 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In, {ArgFlag::canBeMoldNull}},`。
- **L698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultingKIND},`.
  **L698 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultingKIND},`。
- **L699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L699 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"len_trim", {{"string", AnyChar}, DefaultingKIND}, KINDInt},`.
  **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"len_trim", {{"string", AnyChar}, DefaultingKIND}, KINDInt},`。
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"lge", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},`.
  **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"lge", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},`。
- **L702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L702 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"lgt", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},`.
  **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"lgt", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},`。
- **L704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L704 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"lle", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},`.
  **L705 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"lle", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},`。
- **L706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L706 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"llt", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},`.
  **L707 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"llt", {{"string_a", SameCharNoLen}, {"string_b", SameCharNoLen}},`。
- **L708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L708 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"lnblnk", {{"string", AnyChar}}, DefaultInt},`.
  **L709 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"lnblnk", {{"string", AnyChar}}, DefaultInt},`。
- **L710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"loc", {{"x", Addressable, Rank::anyOrAssumedRank}}, SubscriptInt,`.
  **L710 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"loc", {{"x", Addressable, Rank::anyOrAssumedRank}}, SubscriptInt,`。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar},`.
  **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar},`。
- **L712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"log", {{"x", SameFloating}}, SameFloating},`.
  **L712 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"log", {{"x", SameFloating}}, SameFloating},`。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"log10", {{"x", SameReal}}, SameReal},`.
  **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"log10", {{"x", SameReal}}, SameReal},`。
- **L714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"logical", {{"l", AnyLogical}, DefaultingKIND}, KINDLogical},`.
  **L714 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"logical", {{"l", AnyLogical}, DefaultingKIND}, KINDLogical},`。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"log_gamma", {{"x", SameReal}}, SameReal},`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"log_gamma", {{"x", SameReal}}, SameReal},`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"malloc", {{"size", AnyInt}}, SubscriptInt},`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"malloc", {{"size", AnyInt}}, SubscriptInt},`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matmul",`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matmul",`。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"matrix_a", AnyLogical, Rank::vector},`.
  **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"matrix_a", AnyLogical, Rank::vector},`。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matrix_b", AnyLogical, Rank::matrix}},`.
  **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matrix_b", AnyLogical, Rank::matrix}},`。
- **L720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ResultLogical, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L720 CN**: 继续一个多行参数列表、初始化器或聚合项：`ResultLogical, Rank::vector, IntrinsicClass::transformationalFunction},`。

### Lines 721-744

````cpp
    {"matmul",
        {{"matrix_a", AnyLogical, Rank::matrix},
            {"matrix_b", AnyLogical, Rank::vector}},
        ResultLogical, Rank::vector, IntrinsicClass::transformationalFunction},
    {"matmul",
        {{"matrix_a", AnyLogical, Rank::matrix},
            {"matrix_b", AnyLogical, Rank::matrix}},
        ResultLogical, Rank::matrix, IntrinsicClass::transformationalFunction},
    {"matmul",
        {{"matrix_a", AnyNumeric, Rank::vector},
            {"matrix_b", AnyNumeric, Rank::matrix}},
        ResultNumeric, Rank::vector, IntrinsicClass::transformationalFunction},
    {"matmul",
        {{"matrix_a", AnyNumeric, Rank::matrix},
            {"matrix_b", AnyNumeric, Rank::vector}},
        ResultNumeric, Rank::vector, IntrinsicClass::transformationalFunction},
    {"matmul",
        {{"matrix_a", AnyNumeric, Rank::matrix},
            {"matrix_b", AnyNumeric, Rank::matrix}},
        ResultNumeric, Rank::matrix, IntrinsicClass::transformationalFunction},
    {"maskl", {{"i", AnyInt}, DefaultingKIND}, KINDInt},
    {"maskr", {{"i", AnyInt}, DefaultingKIND}, KINDInt},
    {"max",
        {{"a1", OperandIntOrReal}, {"a2", OperandIntOrReal},
````
- **L721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matmul",`.
  **L721 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matmul",`。
- **L722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"matrix_a", AnyLogical, Rank::matrix},`.
  **L722 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"matrix_a", AnyLogical, Rank::matrix},`。
- **L723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matrix_b", AnyLogical, Rank::vector}},`.
  **L723 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matrix_b", AnyLogical, Rank::vector}},`。
- **L724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ResultLogical, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L724 CN**: 继续一个多行参数列表、初始化器或聚合项：`ResultLogical, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matmul",`.
  **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matmul",`。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"matrix_a", AnyLogical, Rank::matrix},`.
  **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"matrix_a", AnyLogical, Rank::matrix},`。
- **L727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matrix_b", AnyLogical, Rank::matrix}},`.
  **L727 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matrix_b", AnyLogical, Rank::matrix}},`。
- **L728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ResultLogical, Rank::matrix, IntrinsicClass::transformationalFunction},`.
  **L728 CN**: 继续一个多行参数列表、初始化器或聚合项：`ResultLogical, Rank::matrix, IntrinsicClass::transformationalFunction},`。
- **L729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matmul",`.
  **L729 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matmul",`。
- **L730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"matrix_a", AnyNumeric, Rank::vector},`.
  **L730 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"matrix_a", AnyNumeric, Rank::vector},`。
- **L731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matrix_b", AnyNumeric, Rank::matrix}},`.
  **L731 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matrix_b", AnyNumeric, Rank::matrix}},`。
- **L732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ResultNumeric, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L732 CN**: 继续一个多行参数列表、初始化器或聚合项：`ResultNumeric, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matmul",`.
  **L733 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matmul",`。
- **L734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"matrix_a", AnyNumeric, Rank::matrix},`.
  **L734 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"matrix_a", AnyNumeric, Rank::matrix},`。
- **L735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matrix_b", AnyNumeric, Rank::vector}},`.
  **L735 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matrix_b", AnyNumeric, Rank::vector}},`。
- **L736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ResultNumeric, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L736 CN**: 继续一个多行参数列表、初始化器或聚合项：`ResultNumeric, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matmul",`.
  **L737 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matmul",`。
- **L738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"matrix_a", AnyNumeric, Rank::matrix},`.
  **L738 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"matrix_a", AnyNumeric, Rank::matrix},`。
- **L739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"matrix_b", AnyNumeric, Rank::matrix}},`.
  **L739 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"matrix_b", AnyNumeric, Rank::matrix}},`。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ResultNumeric, Rank::matrix, IntrinsicClass::transformationalFunction},`.
  **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`ResultNumeric, Rank::matrix, IntrinsicClass::transformationalFunction},`。
- **L741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"maskl", {{"i", AnyInt}, DefaultingKIND}, KINDInt},`.
  **L741 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"maskl", {{"i", AnyInt}, DefaultingKIND}, KINDInt},`。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"maskr", {{"i", AnyInt}, DefaultingKIND}, KINDInt},`.
  **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"maskr", {{"i", AnyInt}, DefaultingKIND}, KINDInt},`。
- **L743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"max",`.
  **L743 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"max",`。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", OperandIntOrReal}, {"a2", OperandIntOrReal},`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", OperandIntOrReal}, {"a2", OperandIntOrReal},`。

### Lines 745-768

````cpp
            {"a3", OperandIntOrReal, Rank::elemental, Optionality::repeats}},
        OperandIntOrReal},
    {"max",
        {{"a1", OperandUnsigned}, {"a2", OperandUnsigned},
            {"a3", OperandUnsigned, Rank::elemental, Optionality::repeats}},
        OperandUnsigned},
    {"max",
        {{"a1", SameCharNoLen}, {"a2", SameCharNoLen},
            {"a3", SameCharNoLen, Rank::elemental, Optionality::repeats}},
        SameCharNoLen},
    {"maxexponent",
        {{"x", AnyReal, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"maxloc",
        {{"array", AnyRelatable, Rank::array}, RequiredDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
        KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},
    {"maxloc",
        {{"array", AnyRelatable, Rank::array}, MissingDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
````
- **L745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", OperandIntOrReal, Rank::elemental, Optionality::repeats}},`.
  **L745 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", OperandIntOrReal, Rank::elemental, Optionality::repeats}},`。
- **L746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandIntOrReal},`.
  **L746 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandIntOrReal},`。
- **L747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"max",`.
  **L747 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"max",`。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", OperandUnsigned}, {"a2", OperandUnsigned},`.
  **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", OperandUnsigned}, {"a2", OperandUnsigned},`。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", OperandUnsigned, Rank::elemental, Optionality::repeats}},`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", OperandUnsigned, Rank::elemental, Optionality::repeats}},`。
- **L750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandUnsigned},`.
  **L750 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandUnsigned},`。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"max",`.
  **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"max",`。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", SameCharNoLen}, {"a2", SameCharNoLen},`.
  **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", SameCharNoLen}, {"a2", SameCharNoLen},`。
- **L753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", SameCharNoLen, Rank::elemental, Optionality::repeats}},`.
  **L753 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", SameCharNoLen, Rank::elemental, Optionality::repeats}},`。
- **L754 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameCharNoLen},`.
  **L754 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameCharNoLen},`。
- **L755 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"maxexponent",`.
  **L755 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"maxexponent",`。
- **L756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::anyOrAssumedRank, Optionality::required,`.
  **L756 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::anyOrAssumedRank, Optionality::required,`。
- **L757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L757 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L758 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L759 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"maxloc",`.
  **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"maxloc",`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyRelatable, Rank::array}, RequiredDIM, OptionalMASK,`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyRelatable, Rank::array}, RequiredDIM, OptionalMASK,`。
- **L762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L762 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`。
- **L765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"maxloc",`.
  **L765 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"maxloc",`。
- **L766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyRelatable, Rank::array}, MissingDIM, OptionalMASK,`.
  **L766 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyRelatable, Rank::array}, MissingDIM, OptionalMASK,`。
- **L767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L767 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L768 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。

### Lines 769-792

````cpp
        KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},
    {"maxval",
        {{"array", SameRelatable, Rank::array}, RequiredDIM, OptionalMASK},
        SameRelatable, Rank::dimReduced,
        IntrinsicClass::transformationalFunction},
    {"maxval",
        {{"array", SameRelatable, Rank::array}, MissingDIM, OptionalMASK},
        SameRelatable, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"merge",
        {{"tsource", SameType}, {"fsource", SameType}, {"mask", AnyLogical}},
        SameType},
    {"merge_bits",
        {{"i", SameIntOrUnsigned},
            {"j", SameIntOrUnsigned, Rank::elementalOrBOZ},
            {"mask", SameIntOrUnsigned, Rank::elementalOrBOZ}},
        SameIntOrUnsigned},
    {"merge_bits",
        {{"i", BOZ}, {"j", SameIntOrUnsigned},
            {"mask", SameIntOrUnsigned, Rank::elementalOrBOZ}},
        SameIntOrUnsigned},
    {"min",
        {{"a1", OperandIntOrReal}, {"a2", OperandIntOrReal},
            {"a3", OperandIntOrReal, Rank::elemental, Optionality::repeats}},
        OperandIntOrReal},
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`。
- **L770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"maxval",`.
  **L770 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"maxval",`。
- **L771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameRelatable, Rank::array}, RequiredDIM, OptionalMASK},`.
  **L771 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameRelatable, Rank::array}, RequiredDIM, OptionalMASK},`。
- **L772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameRelatable, Rank::dimReduced,`.
  **L772 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameRelatable, Rank::dimReduced,`。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"maxval",`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"maxval",`。
- **L775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameRelatable, Rank::array}, MissingDIM, OptionalMASK},`.
  **L775 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameRelatable, Rank::array}, MissingDIM, OptionalMASK},`。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameRelatable, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameRelatable, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"merge",`.
  **L777 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"merge",`。
- **L778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"tsource", SameType}, {"fsource", SameType}, {"mask", AnyLogical}},`.
  **L778 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"tsource", SameType}, {"fsource", SameType}, {"mask", AnyLogical}},`。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType},`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType},`。
- **L780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"merge_bits",`.
  **L780 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"merge_bits",`。
- **L781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", SameIntOrUnsigned},`.
  **L781 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", SameIntOrUnsigned},`。
- **L782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"j", SameIntOrUnsigned, Rank::elementalOrBOZ},`.
  **L782 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"j", SameIntOrUnsigned, Rank::elementalOrBOZ},`。
- **L783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mask", SameIntOrUnsigned, Rank::elementalOrBOZ}},`.
  **L783 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mask", SameIntOrUnsigned, Rank::elementalOrBOZ}},`。
- **L784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L784 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"merge_bits",`.
  **L785 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"merge_bits",`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", BOZ}, {"j", SameIntOrUnsigned},`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", BOZ}, {"j", SameIntOrUnsigned},`。
- **L787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mask", SameIntOrUnsigned, Rank::elementalOrBOZ}},`.
  **L787 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mask", SameIntOrUnsigned, Rank::elementalOrBOZ}},`。
- **L788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L788 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"min",`.
  **L789 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"min",`。
- **L790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", OperandIntOrReal}, {"a2", OperandIntOrReal},`.
  **L790 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", OperandIntOrReal}, {"a2", OperandIntOrReal},`。
- **L791 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", OperandIntOrReal, Rank::elemental, Optionality::repeats}},`.
  **L791 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", OperandIntOrReal, Rank::elemental, Optionality::repeats}},`。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandIntOrReal},`.
  **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandIntOrReal},`。

### Lines 793-816

````cpp
    {"min",
        {{"a1", OperandUnsigned}, {"a2", OperandUnsigned},
            {"a3", OperandUnsigned, Rank::elemental, Optionality::repeats}},
        OperandUnsigned},
    {"min",
        {{"a1", SameCharNoLen}, {"a2", SameCharNoLen},
            {"a3", SameCharNoLen, Rank::elemental, Optionality::repeats}},
        SameCharNoLen},
    {"minexponent",
        {{"x", AnyReal, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"minloc",
        {{"array", AnyRelatable, Rank::array}, RequiredDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
        KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},
    {"minloc",
        {{"array", AnyRelatable, Rank::array}, MissingDIM, OptionalMASK,
            SizeDefaultKIND,
            {"back", AnyLogical, Rank::scalar, Optionality::optional}},
        KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},
    {"minval",
````
- **L793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"min",`.
  **L793 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"min",`。
- **L794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", OperandUnsigned}, {"a2", OperandUnsigned},`.
  **L794 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", OperandUnsigned}, {"a2", OperandUnsigned},`。
- **L795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", OperandUnsigned, Rank::elemental, Optionality::repeats}},`.
  **L795 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", OperandUnsigned, Rank::elemental, Optionality::repeats}},`。
- **L796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandUnsigned},`.
  **L796 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandUnsigned},`。
- **L797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"min",`.
  **L797 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"min",`。
- **L798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", SameCharNoLen}, {"a2", SameCharNoLen},`.
  **L798 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", SameCharNoLen}, {"a2", SameCharNoLen},`。
- **L799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", SameCharNoLen, Rank::elemental, Optionality::repeats}},`.
  **L799 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", SameCharNoLen, Rank::elemental, Optionality::repeats}},`。
- **L800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameCharNoLen},`.
  **L800 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameCharNoLen},`。
- **L801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"minexponent",`.
  **L801 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"minexponent",`。
- **L802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::anyOrAssumedRank, Optionality::required,`.
  **L802 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::anyOrAssumedRank, Optionality::required,`。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L804 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L805 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"minloc",`.
  **L806 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"minloc",`。
- **L807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyRelatable, Rank::array}, RequiredDIM, OptionalMASK,`.
  **L807 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyRelatable, Rank::array}, RequiredDIM, OptionalMASK,`。
- **L808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L808 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L809 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L810 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`.
  **L810 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"minloc",`.
  **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"minloc",`。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyRelatable, Rank::array}, MissingDIM, OptionalMASK,`.
  **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyRelatable, Rank::array}, MissingDIM, OptionalMASK,`。
- **L813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND,`.
  **L813 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND,`。
- **L814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L814 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`.
  **L815 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::locReduced, IntrinsicClass::transformationalFunction},`。
- **L816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"minval",`.
  **L816 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"minval",`。

### Lines 817-840

````cpp
        {{"array", SameRelatable, Rank::array}, RequiredDIM, OptionalMASK},
        SameRelatable, Rank::dimReduced,
        IntrinsicClass::transformationalFunction},
    {"minval",
        {{"array", SameRelatable, Rank::array}, MissingDIM, OptionalMASK},
        SameRelatable, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"mod", {{"a", OperandIntOrReal}, {"p", OperandIntOrReal}},
        OperandIntOrReal},
    {"mod", {{"a", OperandUnsigned}, {"p", OperandUnsigned}}, OperandUnsigned},
    {"modulo", {{"a", OperandIntOrReal}, {"p", OperandIntOrReal}},
        OperandIntOrReal},
    {"modulo", {{"a", OperandUnsigned}, {"p", OperandUnsigned}},
        OperandUnsigned},
    {"nearest", {{"x", SameReal}, {"s", AnyReal}}, SameReal},
    {"new_line",
        {{"a", SameCharNoLen, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        SameCharNoLen, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"nint", {{"a", AnyReal}, DefaultingKIND}, KINDInt},
    {"norm2", {{"x", SameReal, Rank::array}, RequiredDIM}, SameReal,
        Rank::dimReduced, IntrinsicClass::transformationalFunction},
    {"norm2", {{"x", SameReal, Rank::array}, MissingDIM}, SameReal,
        Rank::scalar, IntrinsicClass::transformationalFunction},
````
- **L817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameRelatable, Rank::array}, RequiredDIM, OptionalMASK},`.
  **L817 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameRelatable, Rank::array}, RequiredDIM, OptionalMASK},`。
- **L818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameRelatable, Rank::dimReduced,`.
  **L818 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameRelatable, Rank::dimReduced,`。
- **L819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L819 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"minval",`.
  **L820 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"minval",`。
- **L821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameRelatable, Rank::array}, MissingDIM, OptionalMASK},`.
  **L821 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameRelatable, Rank::array}, MissingDIM, OptionalMASK},`。
- **L822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameRelatable, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L822 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameRelatable, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mod", {{"a", OperandIntOrReal}, {"p", OperandIntOrReal}},`.
  **L823 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mod", {{"a", OperandIntOrReal}, {"p", OperandIntOrReal}},`。
- **L824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandIntOrReal},`.
  **L824 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandIntOrReal},`。
- **L825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mod", {{"a", OperandUnsigned}, {"p", OperandUnsigned}}, OperandUnsigned},`.
  **L825 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mod", {{"a", OperandUnsigned}, {"p", OperandUnsigned}}, OperandUnsigned},`。
- **L826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"modulo", {{"a", OperandIntOrReal}, {"p", OperandIntOrReal}},`.
  **L826 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"modulo", {{"a", OperandIntOrReal}, {"p", OperandIntOrReal}},`。
- **L827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandIntOrReal},`.
  **L827 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandIntOrReal},`。
- **L828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"modulo", {{"a", OperandUnsigned}, {"p", OperandUnsigned}},`.
  **L828 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"modulo", {{"a", OperandUnsigned}, {"p", OperandUnsigned}},`。
- **L829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OperandUnsigned},`.
  **L829 CN**: 继续一个多行参数列表、初始化器或聚合项：`OperandUnsigned},`。
- **L830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"nearest", {{"x", SameReal}, {"s", AnyReal}}, SameReal},`.
  **L830 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"nearest", {{"x", SameReal}, {"s", AnyReal}}, SameReal},`。
- **L831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"new_line",`.
  **L831 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"new_line",`。
- **L832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", SameCharNoLen, Rank::anyOrAssumedRank, Optionality::required,`.
  **L832 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", SameCharNoLen, Rank::anyOrAssumedRank, Optionality::required,`。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameCharNoLen, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L835 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameCharNoLen, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"nint", {{"a", AnyReal}, DefaultingKIND}, KINDInt},`.
  **L836 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"nint", {{"a", AnyReal}, DefaultingKIND}, KINDInt},`。
- **L837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"norm2", {{"x", SameReal, Rank::array}, RequiredDIM}, SameReal,`.
  **L837 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"norm2", {{"x", SameReal, Rank::array}, RequiredDIM}, SameReal,`。
- **L838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::dimReduced, IntrinsicClass::transformationalFunction},`.
  **L838 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::dimReduced, IntrinsicClass::transformationalFunction},`。
- **L839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"norm2", {{"x", SameReal, Rank::array}, MissingDIM}, SameReal,`.
  **L839 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"norm2", {{"x", SameReal, Rank::array}, MissingDIM}, SameReal,`。
- **L840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L840 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::transformationalFunction},`。

### Lines 841-864

````cpp
    {"not", {{"i", SameIntOrUnsigned}}, SameIntOrUnsigned},
    // NULL() is a special case handled in Probe() below
    {"num_images", {}, DefaultInt, Rank::scalar,
        IntrinsicClass::transformationalFunction},
    {"num_images", {{"team", TeamType, Rank::scalar}}, DefaultInt, Rank::scalar,
        IntrinsicClass::transformationalFunction},
    {"num_images", {{"team_number", AnyInt, Rank::scalar}}, DefaultInt,
        Rank::scalar, IntrinsicClass::transformationalFunction},
    {"out_of_range",
        {{"x", AnyIntOrReal}, {"mold", AnyIntOrReal, Rank::scalar}},
        DefaultLogical},
    {"out_of_range",
        {{"x", AnyReal}, {"mold", AnyInt, Rank::scalar},
            {"round", AnyLogical, Rank::scalar, Optionality::optional}},
        DefaultLogical},
    {"out_of_range", {{"x", AnyReal}, {"mold", AnyReal}}, DefaultLogical},
    {"pack",
        {{"array", SameType, Rank::array},
            {"mask", AnyLogical, Rank::conformable},
            {"vector", SameType, Rank::vector, Optionality::optional}},
        SameType, Rank::vector, IntrinsicClass::transformationalFunction},
    {"parity", {{"mask", SameLogical, Rank::array}, OptionalDIM}, SameLogical,
        Rank::dimReduced, IntrinsicClass::transformationalFunction},
    {"popcnt", {{"i", AnyInt}}, DefaultInt},
````
- **L841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"not", {{"i", SameIntOrUnsigned}}, SameIntOrUnsigned},`.
  **L841 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"not", {{"i", SameIntOrUnsigned}}, SameIntOrUnsigned},`。
- **L842 EN**: Comment explains nearby logic, intent, or metadata: `NULL() is a special case handled in Probe() below`.
  **L842 CN**: 注释说明附近代码的逻辑、意图或元数据：`NULL() is a special case handled in Probe() below`。
- **L843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"num_images", {}, DefaultInt, Rank::scalar,`.
  **L843 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"num_images", {}, DefaultInt, Rank::scalar,`。
- **L844 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L844 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L845 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"num_images", {{"team", TeamType, Rank::scalar}}, DefaultInt, Rank::scalar,`.
  **L845 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"num_images", {{"team", TeamType, Rank::scalar}}, DefaultInt, Rank::scalar,`。
- **L846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L846 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"num_images", {{"team_number", AnyInt, Rank::scalar}}, DefaultInt,`.
  **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"num_images", {{"team_number", AnyInt, Rank::scalar}}, DefaultInt,`。
- **L848 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L848 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"out_of_range",`.
  **L849 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"out_of_range",`。
- **L850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyIntOrReal}, {"mold", AnyIntOrReal, Rank::scalar}},`.
  **L850 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyIntOrReal}, {"mold", AnyIntOrReal, Rank::scalar}},`。
- **L851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L851 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"out_of_range",`.
  **L852 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"out_of_range",`。
- **L853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal}, {"mold", AnyInt, Rank::scalar},`.
  **L853 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal}, {"mold", AnyInt, Rank::scalar},`。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"round", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"round", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L855 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"out_of_range", {{"x", AnyReal}, {"mold", AnyReal}}, DefaultLogical},`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"out_of_range", {{"x", AnyReal}, {"mold", AnyReal}}, DefaultLogical},`。
- **L857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pack",`.
  **L857 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pack",`。
- **L858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameType, Rank::array},`.
  **L858 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameType, Rank::array},`。
- **L859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mask", AnyLogical, Rank::conformable},`.
  **L859 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mask", AnyLogical, Rank::conformable},`。
- **L860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"vector", SameType, Rank::vector, Optionality::optional}},`.
  **L860 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"vector", SameType, Rank::vector, Optionality::optional}},`。
- **L861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L861 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"parity", {{"mask", SameLogical, Rank::array}, OptionalDIM}, SameLogical,`.
  **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"parity", {{"mask", SameLogical, Rank::array}, OptionalDIM}, SameLogical,`。
- **L863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::dimReduced, IntrinsicClass::transformationalFunction},`.
  **L863 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::dimReduced, IntrinsicClass::transformationalFunction},`。
- **L864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"popcnt", {{"i", AnyInt}}, DefaultInt},`.
  **L864 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"popcnt", {{"i", AnyInt}}, DefaultInt},`。

### Lines 865-888

````cpp
    {"poppar", {{"i", AnyInt}}, DefaultInt},
    {"product",
        {{"array", SameNumeric, Rank::array}, RequiredDIM, OptionalMASK},
        SameNumeric, Rank::dimReduced,
        IntrinsicClass::transformationalFunction},
    {"product", {{"array", SameNumeric, Rank::array}, MissingDIM, OptionalMASK},
        SameNumeric, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"precision",
        {{"x", AnyFloating, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"present", {{"a", Addressable, Rank::anyOrAssumedRank}}, DefaultLogical,
        Rank::scalar, IntrinsicClass::inquiryFunction},
    {"putenv", {{"str", DefaultChar, Rank::scalar}}, DefaultInt, Rank::scalar,
        IntrinsicClass::transformationalFunction},
    {"radix",
        {{"x", AnyIntOrReal, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"rand",
        {{"i", TypePattern{IntType, KindCode::exactKind, 4}, Rank::scalar,
            Optionality::optional}},
````
- **L865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"poppar", {{"i", AnyInt}}, DefaultInt},`.
  **L865 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"poppar", {{"i", AnyInt}}, DefaultInt},`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"product",`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"product",`。
- **L867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameNumeric, Rank::array}, RequiredDIM, OptionalMASK},`.
  **L867 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameNumeric, Rank::array}, RequiredDIM, OptionalMASK},`。
- **L868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameNumeric, Rank::dimReduced,`.
  **L868 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameNumeric, Rank::dimReduced,`。
- **L869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L869 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"product", {{"array", SameNumeric, Rank::array}, MissingDIM, OptionalMASK},`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"product", {{"array", SameNumeric, Rank::array}, MissingDIM, OptionalMASK},`。
- **L871 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameNumeric, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L871 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameNumeric, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L872 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"precision",`.
  **L872 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"precision",`。
- **L873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyFloating, Rank::anyOrAssumedRank, Optionality::required,`.
  **L873 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyFloating, Rank::anyOrAssumedRank, Optionality::required,`。
- **L874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L874 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L875 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L876 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L876 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"present", {{"a", Addressable, Rank::anyOrAssumedRank}}, DefaultLogical,`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"present", {{"a", Addressable, Rank::anyOrAssumedRank}}, DefaultLogical,`。
- **L878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L878 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"putenv", {{"str", DefaultChar, Rank::scalar}}, DefaultInt, Rank::scalar,`.
  **L879 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"putenv", {{"str", DefaultChar, Rank::scalar}}, DefaultInt, Rank::scalar,`。
- **L880 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L880 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L881 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"radix",`.
  **L881 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"radix",`。
- **L882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyIntOrReal, Rank::anyOrAssumedRank, Optionality::required,`.
  **L882 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyIntOrReal, Rank::anyOrAssumedRank, Optionality::required,`。
- **L883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L883 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L885 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"rand",`.
  **L886 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"rand",`。
- **L887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"i", TypePattern{IntType, KindCode::exactKind, 4}, Rank::scalar,`.
  **L887 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"i", TypePattern{IntType, KindCode::exactKind, 4}, Rank::scalar,`。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::optional}},`.
  **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::optional}},`。

### Lines 889-912

````cpp
        TypePattern{RealType, KindCode::exactKind, 4}, Rank::scalar,
        IntrinsicClass::impureFunction},
    {"range",
        {{"x", AnyNumeric, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"rank",
        {{"a", AnyData, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"real", {{"a", SameComplex, Rank::elemental}},
        SameReal}, // 16.9.160(4)(ii)
    {"real", {{"a", AnyNumeric, Rank::elementalOrBOZ}, DefaultingKIND},
        KINDReal},
    {"reduce",
        {{"array", SameType, Rank::array},
            {"operation", SameType, Rank::reduceOperation}, RequiredDIM,
            OptionalMASK,
            {"identity", SameType, Rank::scalar, Optionality::optional},
            {"ordered", AnyLogical, Rank::scalar, Optionality::optional}},
        SameType, Rank::dimReduced, IntrinsicClass::transformationalFunction},
    {"reduce",
````
- **L889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{RealType, KindCode::exactKind, 4}, Rank::scalar,`.
  **L889 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{RealType, KindCode::exactKind, 4}, Rank::scalar,`。
- **L890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::impureFunction},`.
  **L890 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::impureFunction},`。
- **L891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"range",`.
  **L891 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"range",`。
- **L892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyNumeric, Rank::anyOrAssumedRank, Optionality::required,`.
  **L892 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyNumeric, Rank::anyOrAssumedRank, Optionality::required,`。
- **L893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L893 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L894 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L895 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"rank",`.
  **L896 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"rank",`。
- **L897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", AnyData, Rank::anyOrAssumedRank, Optionality::required,`.
  **L897 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", AnyData, Rank::anyOrAssumedRank, Optionality::required,`。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L900 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"real", {{"a", SameComplex, Rank::elemental}},`.
  **L901 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"real", {{"a", SameComplex, Rank::elemental}},`。
- **L902 EN**: Continues the surrounding expression or declaration: `SameReal}, // 16.9.160(4)(ii)`.
  **L902 CN**: 继续构造周围的表达式或声明：`SameReal}, // 16.9.160(4)(ii)`。
- **L903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"real", {{"a", AnyNumeric, Rank::elementalOrBOZ}, DefaultingKIND},`.
  **L903 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"real", {{"a", AnyNumeric, Rank::elementalOrBOZ}, DefaultingKIND},`。
- **L904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDReal},`.
  **L904 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDReal},`。
- **L905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"reduce",`.
  **L905 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"reduce",`。
- **L906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameType, Rank::array},`.
  **L906 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameType, Rank::array},`。
- **L907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"operation", SameType, Rank::reduceOperation}, RequiredDIM,`.
  **L907 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"operation", SameType, Rank::reduceOperation}, RequiredDIM,`。
- **L908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OptionalMASK,`.
  **L908 CN**: 继续一个多行参数列表、初始化器或聚合项：`OptionalMASK,`。
- **L909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"identity", SameType, Rank::scalar, Optionality::optional},`.
  **L909 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"identity", SameType, Rank::scalar, Optionality::optional},`。
- **L910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ordered", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L910 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ordered", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L911 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::dimReduced, IntrinsicClass::transformationalFunction},`.
  **L911 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::dimReduced, IntrinsicClass::transformationalFunction},`。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"reduce",`.
  **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"reduce",`。

### Lines 913-936

````cpp
        {{"array", SameType, Rank::array},
            {"operation", SameType, Rank::reduceOperation}, MissingDIM,
            OptionalMASK,
            {"identity", SameType, Rank::scalar, Optionality::optional},
            {"ordered", AnyLogical, Rank::scalar, Optionality::optional}},
        SameType, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"rename",
        {{"path1", DefaultChar, Rank::scalar},
            {"path2", DefaultChar, Rank::scalar}},
        DefaultInt, Rank::scalar},
    {"repeat",
        {{"string", SameCharNoLen, Rank::scalar},
            {"ncopies", AnyInt, Rank::scalar}},
        SameCharNoLen, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"reshape",
        {{"source", SameType, Rank::array}, {"shape", AnyInt, Rank::shape},
            {"pad", SameType, Rank::array, Optionality::optional},
            {"order", AnyInt, Rank::vector, Optionality::optional}},
        SameType, Rank::shaped, IntrinsicClass::transformationalFunction},
    {"rrspacing", {{"x", SameReal}}, SameReal},
    {"rtc", {}, TypePattern{RealType, KindCode::exactKind, 8}, Rank::scalar},
    {"same_type_as",
        {{"a", ExtensibleDerived, Rank::anyOrAssumedRank, Optionality::required,
             common::Intent::In, {ArgFlag::canBeMoldNull}},
````
- **L913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", SameType, Rank::array},`.
  **L913 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", SameType, Rank::array},`。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"operation", SameType, Rank::reduceOperation}, MissingDIM,`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"operation", SameType, Rank::reduceOperation}, MissingDIM,`。
- **L915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OptionalMASK,`.
  **L915 CN**: 继续一个多行参数列表、初始化器或聚合项：`OptionalMASK,`。
- **L916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"identity", SameType, Rank::scalar, Optionality::optional},`.
  **L916 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"identity", SameType, Rank::scalar, Optionality::optional},`。
- **L917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ordered", AnyLogical, Rank::scalar, Optionality::optional}},`.
  **L917 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ordered", AnyLogical, Rank::scalar, Optionality::optional}},`。
- **L918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L918 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"rename",`.
  **L919 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"rename",`。
- **L920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"path1", DefaultChar, Rank::scalar},`.
  **L920 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"path1", DefaultChar, Rank::scalar},`。
- **L921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"path2", DefaultChar, Rank::scalar}},`.
  **L921 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"path2", DefaultChar, Rank::scalar}},`。
- **L922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar},`.
  **L922 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar},`。
- **L923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"repeat",`.
  **L923 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"repeat",`。
- **L924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"string", SameCharNoLen, Rank::scalar},`.
  **L924 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"string", SameCharNoLen, Rank::scalar},`。
- **L925 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ncopies", AnyInt, Rank::scalar}},`.
  **L925 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ncopies", AnyInt, Rank::scalar}},`。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameCharNoLen, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameCharNoLen, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"reshape",`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"reshape",`。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"source", SameType, Rank::array}, {"shape", AnyInt, Rank::shape},`.
  **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"source", SameType, Rank::array}, {"shape", AnyInt, Rank::shape},`。
- **L929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pad", SameType, Rank::array, Optionality::optional},`.
  **L929 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pad", SameType, Rank::array, Optionality::optional},`。
- **L930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"order", AnyInt, Rank::vector, Optionality::optional}},`.
  **L930 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"order", AnyInt, Rank::vector, Optionality::optional}},`。
- **L931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::shaped, IntrinsicClass::transformationalFunction},`.
  **L931 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::shaped, IntrinsicClass::transformationalFunction},`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"rrspacing", {{"x", SameReal}}, SameReal},`.
  **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"rrspacing", {{"x", SameReal}}, SameReal},`。
- **L933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"rtc", {}, TypePattern{RealType, KindCode::exactKind, 8}, Rank::scalar},`.
  **L933 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"rtc", {}, TypePattern{RealType, KindCode::exactKind, 8}, Rank::scalar},`。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"same_type_as",`.
  **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"same_type_as",`。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", ExtensibleDerived, Rank::anyOrAssumedRank, Optionality::required,`.
  **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", ExtensibleDerived, Rank::anyOrAssumedRank, Optionality::required,`。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In, {ArgFlag::canBeMoldNull}},`.
  **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In, {ArgFlag::canBeMoldNull}},`。

### Lines 937-960

````cpp
            {"b", ExtensibleDerived, Rank::anyOrAssumedRank,
                Optionality::required, common::Intent::In,
                {ArgFlag::canBeMoldNull}}},
        DefaultLogical, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"scale", {{"x", SameReal}, {"i", AnyInt}}, SameReal}, // == IEEE_SCALB()
    {"scan",
        {{"string", SameCharNoLen}, {"set", SameCharNoLen},
            {"back", AnyLogical, Rank::elemental, Optionality::optional},
            DefaultingKIND},
        KINDInt},
    {"secnds",
        {{"refTime", TypePattern{RealType, KindCode::exactKind, 4},
            Rank::scalar}},
        TypePattern{RealType, KindCode::exactKind, 4}, Rank::scalar},
    {"second", {}, DefaultReal, Rank::scalar},
    {"selected_char_kind", {{"name", DefaultChar, Rank::scalar}}, DefaultInt,
        Rank::scalar, IntrinsicClass::transformationalFunction},
    {"selected_int_kind", {{"r", AnyInt, Rank::scalar}}, DefaultInt,
        Rank::scalar, IntrinsicClass::transformationalFunction},
    {"selected_logical_kind", {{"bits", AnyInt, Rank::scalar}}, DefaultInt,
        Rank::scalar, IntrinsicClass::transformationalFunction},
    {"selected_real_kind",
        {{"p", AnyInt, Rank::scalar},
            {"r", AnyInt, Rank::scalar, Optionality::optional},
````
- **L937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", ExtensibleDerived, Rank::anyOrAssumedRank,`.
  **L937 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", ExtensibleDerived, Rank::anyOrAssumedRank,`。
- **L938 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::required, common::Intent::In,`.
  **L938 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::required, common::Intent::In,`。
- **L939 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull}}},`.
  **L939 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull}}},`。
- **L940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L940 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L941 EN**: Continues logic associated with callable symbol `IEEE_SCALB`.
  **L941 CN**: 继续与可调用符号 `IEEE_SCALB` 相关的逻辑。
- **L942 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"scan",`.
  **L942 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"scan",`。
- **L943 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"string", SameCharNoLen}, {"set", SameCharNoLen},`.
  **L943 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"string", SameCharNoLen}, {"set", SameCharNoLen},`。
- **L944 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::elemental, Optionality::optional},`.
  **L944 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::elemental, Optionality::optional},`。
- **L945 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultingKIND},`.
  **L945 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultingKIND},`。
- **L946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt},`.
  **L946 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt},`。
- **L947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"secnds",`.
  **L947 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"secnds",`。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"refTime", TypePattern{RealType, KindCode::exactKind, 4},`.
  **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"refTime", TypePattern{RealType, KindCode::exactKind, 4},`。
- **L949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar}},`.
  **L949 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar}},`。
- **L950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{RealType, KindCode::exactKind, 4}, Rank::scalar},`.
  **L950 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{RealType, KindCode::exactKind, 4}, Rank::scalar},`。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"second", {}, DefaultReal, Rank::scalar},`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"second", {}, DefaultReal, Rank::scalar},`。
- **L952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"selected_char_kind", {{"name", DefaultChar, Rank::scalar}}, DefaultInt,`.
  **L952 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"selected_char_kind", {{"name", DefaultChar, Rank::scalar}}, DefaultInt,`。
- **L953 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L953 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"selected_int_kind", {{"r", AnyInt, Rank::scalar}}, DefaultInt,`.
  **L954 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"selected_int_kind", {{"r", AnyInt, Rank::scalar}}, DefaultInt,`。
- **L955 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L955 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"selected_logical_kind", {{"bits", AnyInt, Rank::scalar}}, DefaultInt,`.
  **L956 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"selected_logical_kind", {{"bits", AnyInt, Rank::scalar}}, DefaultInt,`。
- **L957 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L957 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L958 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"selected_real_kind",`.
  **L958 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"selected_real_kind",`。
- **L959 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"p", AnyInt, Rank::scalar},`.
  **L959 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"p", AnyInt, Rank::scalar},`。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"r", AnyInt, Rank::scalar, Optionality::optional},`.
  **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"r", AnyInt, Rank::scalar, Optionality::optional},`。

### Lines 961-984

````cpp
            {"radix", AnyInt, Rank::scalar, Optionality::optional}},
        DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"selected_real_kind",
        {{"p", AnyInt, Rank::scalar, Optionality::optional},
            {"r", AnyInt, Rank::scalar},
            {"radix", AnyInt, Rank::scalar, Optionality::optional}},
        DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"selected_real_kind",
        {{"p", AnyInt, Rank::scalar, Optionality::optional},
            {"r", AnyInt, Rank::scalar, Optionality::optional},
            {"radix", AnyInt, Rank::scalar}},
        DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"selected_unsigned_kind", {{"r", AnyInt, Rank::scalar}}, DefaultInt,
        Rank::scalar, IntrinsicClass::transformationalFunction},
    {"set_exponent", {{"x", SameReal}, {"i", AnyInt}}, SameReal},
    {"shape", {{"source", AnyData, Rank::anyOrAssumedRank}, SizeDefaultKIND},
        KINDInt, Rank::vector, IntrinsicClass::inquiryFunction},
    {"shifta", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}},
        SameIntOrUnsigned},
    {"shiftl", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}},
        SameIntOrUnsigned},
    {"shiftr", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}},
        SameIntOrUnsigned},
    {"sign", {{"a", SameInt}, {"b", AnyInt}}, SameInt},
````
- **L961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"radix", AnyInt, Rank::scalar, Optionality::optional}},`.
  **L961 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"radix", AnyInt, Rank::scalar, Optionality::optional}},`。
- **L962 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L962 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"selected_real_kind",`.
  **L963 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"selected_real_kind",`。
- **L964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"p", AnyInt, Rank::scalar, Optionality::optional},`.
  **L964 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"p", AnyInt, Rank::scalar, Optionality::optional},`。
- **L965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"r", AnyInt, Rank::scalar},`.
  **L965 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"r", AnyInt, Rank::scalar},`。
- **L966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"radix", AnyInt, Rank::scalar, Optionality::optional}},`.
  **L966 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"radix", AnyInt, Rank::scalar, Optionality::optional}},`。
- **L967 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L967 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"selected_real_kind",`.
  **L968 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"selected_real_kind",`。
- **L969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"p", AnyInt, Rank::scalar, Optionality::optional},`.
  **L969 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"p", AnyInt, Rank::scalar, Optionality::optional},`。
- **L970 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"r", AnyInt, Rank::scalar, Optionality::optional},`.
  **L970 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"r", AnyInt, Rank::scalar, Optionality::optional},`。
- **L971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"radix", AnyInt, Rank::scalar}},`.
  **L971 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"radix", AnyInt, Rank::scalar}},`。
- **L972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L972 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"selected_unsigned_kind", {{"r", AnyInt, Rank::scalar}}, DefaultInt,`.
  **L973 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"selected_unsigned_kind", {{"r", AnyInt, Rank::scalar}}, DefaultInt,`。
- **L974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L974 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"set_exponent", {{"x", SameReal}, {"i", AnyInt}}, SameReal},`.
  **L975 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"set_exponent", {{"x", SameReal}, {"i", AnyInt}}, SameReal},`。
- **L976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"shape", {{"source", AnyData, Rank::anyOrAssumedRank}, SizeDefaultKIND},`.
  **L976 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"shape", {{"source", AnyData, Rank::anyOrAssumedRank}, SizeDefaultKIND},`。
- **L977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::vector, IntrinsicClass::inquiryFunction},`.
  **L977 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::vector, IntrinsicClass::inquiryFunction},`。
- **L978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"shifta", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}},`.
  **L978 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"shifta", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}},`。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L980 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"shiftl", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}},`.
  **L980 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"shiftl", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}},`。
- **L981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L981 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"shiftr", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}},`.
  **L982 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"shiftr", {{"i", SameIntOrUnsigned}, {"shift", AnyInt}},`。
- **L983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameIntOrUnsigned},`.
  **L983 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameIntOrUnsigned},`。
- **L984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sign", {{"a", SameInt}, {"b", AnyInt}}, SameInt},`.
  **L984 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sign", {{"a", SameInt}, {"b", AnyInt}}, SameInt},`。

### Lines 985-1008

````cpp
    {"sign", {{"a", SameReal}, {"b", AnyReal}}, SameReal},
    {"sin", {{"x", SameFloating}}, SameFloating},
    {"sind", {{"x", SameFloating}}, SameFloating},
    {"sinh", {{"x", SameFloating}}, SameFloating},
    {"sinpi", {{"x", SameFloating}}, SameFloating},
    {"size",
        {{"array", AnyData, Rank::arrayOrAssumedRank},
            OptionalDIM, // unless array is assumed-size
            SizeDefaultKIND},
        KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"sizeof", {{"x", AnyData, Rank::anyOrAssumedRank}}, SubscriptInt,
        Rank::scalar, IntrinsicClass::inquiryFunction},
    {"spacing", {{"x", SameReal}}, SameReal},
    {"spread",
        {{"source", SameType, Rank::known, Optionality::required,
             common::Intent::In, {ArgFlag::notAssumedSize}},
            RequiredDIM, {"ncopies", AnyInt, Rank::scalar}},
        SameType, Rank::rankPlus1, IntrinsicClass::transformationalFunction},
    {"sqrt", {{"x", SameFloating}}, SameFloating},
    {"stopped_images", {OptionalTEAM, SizeDefaultKIND}, KINDInt, Rank::vector,
        IntrinsicClass::transformationalFunction},
    {"storage_size",
        {{"a", AnyData, Rank::anyOrAssumedRank, Optionality::required,
             common::Intent::In, {ArgFlag::canBeMoldNull}},
````
- **L985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sign", {{"a", SameReal}, {"b", AnyReal}}, SameReal},`.
  **L985 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sign", {{"a", SameReal}, {"b", AnyReal}}, SameReal},`。
- **L986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sin", {{"x", SameFloating}}, SameFloating},`.
  **L986 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sin", {{"x", SameFloating}}, SameFloating},`。
- **L987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sind", {{"x", SameFloating}}, SameFloating},`.
  **L987 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sind", {{"x", SameFloating}}, SameFloating},`。
- **L988 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sinh", {{"x", SameFloating}}, SameFloating},`.
  **L988 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sinh", {{"x", SameFloating}}, SameFloating},`。
- **L989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sinpi", {{"x", SameFloating}}, SameFloating},`.
  **L989 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sinpi", {{"x", SameFloating}}, SameFloating},`。
- **L990 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"size",`.
  **L990 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"size",`。
- **L991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyData, Rank::arrayOrAssumedRank},`.
  **L991 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyData, Rank::arrayOrAssumedRank},`。
- **L992 EN**: Continues the surrounding expression or declaration: `OptionalDIM, // unless array is assumed-size`.
  **L992 CN**: 继续构造周围的表达式或声明：`OptionalDIM, // unless array is assumed-size`。
- **L993 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND},`.
  **L993 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND},`。
- **L994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L994 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L995 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sizeof", {{"x", AnyData, Rank::anyOrAssumedRank}}, SubscriptInt,`.
  **L995 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sizeof", {{"x", AnyData, Rank::anyOrAssumedRank}}, SubscriptInt,`。
- **L996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L996 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"spacing", {{"x", SameReal}}, SameReal},`.
  **L997 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"spacing", {{"x", SameReal}}, SameReal},`。
- **L998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"spread",`.
  **L998 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"spread",`。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"source", SameType, Rank::known, Optionality::required,`.
  **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"source", SameType, Rank::known, Optionality::required,`。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In, {ArgFlag::notAssumedSize}},`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In, {ArgFlag::notAssumedSize}},`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RequiredDIM, {"ncopies", AnyInt, Rank::scalar}},`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`RequiredDIM, {"ncopies", AnyInt, Rank::scalar}},`。
- **L1002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::rankPlus1, IntrinsicClass::transformationalFunction},`.
  **L1002 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::rankPlus1, IntrinsicClass::transformationalFunction},`。
- **L1003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sqrt", {{"x", SameFloating}}, SameFloating},`.
  **L1003 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sqrt", {{"x", SameFloating}}, SameFloating},`。
- **L1004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stopped_images", {OptionalTEAM, SizeDefaultKIND}, KINDInt, Rank::vector,`.
  **L1004 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stopped_images", {OptionalTEAM, SizeDefaultKIND}, KINDInt, Rank::vector,`。
- **L1005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L1005 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L1006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"storage_size",`.
  **L1006 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"storage_size",`。
- **L1007 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", AnyData, Rank::anyOrAssumedRank, Optionality::required,`.
  **L1007 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", AnyData, Rank::anyOrAssumedRank, Optionality::required,`。
- **L1008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In, {ArgFlag::canBeMoldNull}},`.
  **L1008 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In, {ArgFlag::canBeMoldNull}},`。

### Lines 1009-1032

````cpp
            SizeDefaultKIND},
        KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"sum", {{"array", SameNumeric, Rank::array}, RequiredDIM, OptionalMASK},
        SameNumeric, Rank::dimReduced,
        IntrinsicClass::transformationalFunction},
    {"sum", {{"array", SameNumeric, Rank::array}, MissingDIM, OptionalMASK},
        SameNumeric, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"system", {{"command", DefaultChar, Rank::scalar}}, DefaultInt,
        Rank::scalar},
    {"tan", {{"x", SameFloating}}, SameFloating},
    {"tand", {{"x", SameFloating}}, SameFloating},
    {"tanh", {{"x", SameFloating}}, SameFloating},
    {"tanpi", {{"x", SameFloating}}, SameFloating},
    {"team_number", {OptionalTEAM}, DefaultInt, Rank::scalar,
        IntrinsicClass::transformationalFunction},
    {"this_image",
        {{"coarray", AnyData, Rank::coarray}, RequiredDIM, OptionalTEAM},
        DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"this_image", {{"coarray", AnyData, Rank::coarray}, OptionalTEAM},
        DefaultInt, Rank::vector, IntrinsicClass::transformationalFunction},
    {"this_image", {OptionalTEAM}, DefaultInt, Rank::scalar,
        IntrinsicClass::transformationalFunction},
    {"time", {}, TypePattern{IntType, KindCode::exactKind, 8}, Rank::scalar,
        IntrinsicClass::transformationalFunction},
````
- **L1009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND},`.
  **L1009 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND},`。
- **L1010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L1010 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L1011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sum", {{"array", SameNumeric, Rank::array}, RequiredDIM, OptionalMASK},`.
  **L1011 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sum", {{"array", SameNumeric, Rank::array}, RequiredDIM, OptionalMASK},`。
- **L1012 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameNumeric, Rank::dimReduced,`.
  **L1012 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameNumeric, Rank::dimReduced,`。
- **L1013 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L1013 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L1014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sum", {{"array", SameNumeric, Rank::array}, MissingDIM, OptionalMASK},`.
  **L1014 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sum", {{"array", SameNumeric, Rank::array}, MissingDIM, OptionalMASK},`。
- **L1015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameNumeric, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L1015 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameNumeric, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L1016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"system", {{"command", DefaultChar, Rank::scalar}}, DefaultInt,`.
  **L1016 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"system", {{"command", DefaultChar, Rank::scalar}}, DefaultInt,`。
- **L1017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar},`.
  **L1017 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar},`。
- **L1018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"tan", {{"x", SameFloating}}, SameFloating},`.
  **L1018 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"tan", {{"x", SameFloating}}, SameFloating},`。
- **L1019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"tand", {{"x", SameFloating}}, SameFloating},`.
  **L1019 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"tand", {{"x", SameFloating}}, SameFloating},`。
- **L1020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"tanh", {{"x", SameFloating}}, SameFloating},`.
  **L1020 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"tanh", {{"x", SameFloating}}, SameFloating},`。
- **L1021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"tanpi", {{"x", SameFloating}}, SameFloating},`.
  **L1021 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"tanpi", {{"x", SameFloating}}, SameFloating},`。
- **L1022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"team_number", {OptionalTEAM}, DefaultInt, Rank::scalar,`.
  **L1022 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"team_number", {OptionalTEAM}, DefaultInt, Rank::scalar,`。
- **L1023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L1023 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L1024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"this_image",`.
  **L1024 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"this_image",`。
- **L1025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"coarray", AnyData, Rank::coarray}, RequiredDIM, OptionalTEAM},`.
  **L1025 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"coarray", AnyData, Rank::coarray}, RequiredDIM, OptionalTEAM},`。
- **L1026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L1026 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L1027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"this_image", {{"coarray", AnyData, Rank::coarray}, OptionalTEAM},`.
  **L1027 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"this_image", {{"coarray", AnyData, Rank::coarray}, OptionalTEAM},`。
- **L1028 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L1028 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L1029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"this_image", {OptionalTEAM}, DefaultInt, Rank::scalar,`.
  **L1029 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"this_image", {OptionalTEAM}, DefaultInt, Rank::scalar,`。
- **L1030 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L1030 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L1031 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"time", {}, TypePattern{IntType, KindCode::exactKind, 8}, Rank::scalar,`.
  **L1031 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"time", {}, TypePattern{IntType, KindCode::exactKind, 8}, Rank::scalar,`。
- **L1032 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L1032 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。

### Lines 1033-1056

````cpp
    {"timef", {}, TypePattern{RealType, KindCode::exactKind, 8}, Rank::scalar,
        IntrinsicClass::transformationalFunction},
    {"tiny",
        {{"x", SameReal, Rank::anyOrAssumedRank, Optionality::required,
            common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        SameReal, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"trailz", {{"i", AnyInt}}, DefaultInt},
    {"transfer",
        {{"source", AnyData, Rank::known}, {"mold", SameType, Rank::scalar}},
        SameType, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"transfer",
        {{"source", AnyData, Rank::known}, {"mold", SameType, Rank::array}},
        SameType, Rank::vector, IntrinsicClass::transformationalFunction},
    {"transfer",
        {{"source", AnyData, Rank::anyOrAssumedRank},
            {"mold", SameType, Rank::anyOrAssumedRank},
            {"size", AnyInt, Rank::scalar}},
        SameType, Rank::vector, IntrinsicClass::transformationalFunction},
    // TRANSFER(BOZ, MOLD=integer or real scalar) extension
    {"transfer",
        {{"source", AnyNumeric, Rank::elementalOrBOZ},
            {"mold", SameInt, Rank::scalar}},
        SameInt, Rank::scalar, IntrinsicClass::transformationalFunction},
````
- **L1033 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"timef", {}, TypePattern{RealType, KindCode::exactKind, 8}, Rank::scalar,`.
  **L1033 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"timef", {}, TypePattern{RealType, KindCode::exactKind, 8}, Rank::scalar,`。
- **L1034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L1034 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L1035 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"tiny",`.
  **L1035 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"tiny",`。
- **L1036 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", SameReal, Rank::anyOrAssumedRank, Optionality::required,`.
  **L1036 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", SameReal, Rank::anyOrAssumedRank, Optionality::required,`。
- **L1037 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L1037 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L1038 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1038 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameReal, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L1039 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameReal, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L1040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"trailz", {{"i", AnyInt}}, DefaultInt},`.
  **L1040 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"trailz", {{"i", AnyInt}}, DefaultInt},`。
- **L1041 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"transfer",`.
  **L1041 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"transfer",`。
- **L1042 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"source", AnyData, Rank::known}, {"mold", SameType, Rank::scalar}},`.
  **L1042 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"source", AnyData, Rank::known}, {"mold", SameType, Rank::scalar}},`。
- **L1043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L1043 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L1044 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"transfer",`.
  **L1044 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"transfer",`。
- **L1045 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"source", AnyData, Rank::known}, {"mold", SameType, Rank::array}},`.
  **L1045 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"source", AnyData, Rank::known}, {"mold", SameType, Rank::array}},`。
- **L1046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L1046 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L1047 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"transfer",`.
  **L1047 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"transfer",`。
- **L1048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"source", AnyData, Rank::anyOrAssumedRank},`.
  **L1048 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"source", AnyData, Rank::anyOrAssumedRank},`。
- **L1049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mold", SameType, Rank::anyOrAssumedRank},`.
  **L1049 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mold", SameType, Rank::anyOrAssumedRank},`。
- **L1050 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"size", AnyInt, Rank::scalar}},`.
  **L1050 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"size", AnyInt, Rank::scalar}},`。
- **L1051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::vector, IntrinsicClass::transformationalFunction},`.
  **L1051 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::vector, IntrinsicClass::transformationalFunction},`。
- **L1052 EN**: Comment explains nearby logic, intent, or metadata: `TRANSFER(BOZ, MOLD=integer or real scalar) extension`.
  **L1052 CN**: 注释说明附近代码的逻辑、意图或元数据：`TRANSFER(BOZ, MOLD=integer or real scalar) extension`。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"transfer",`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"transfer",`。
- **L1054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"source", AnyNumeric, Rank::elementalOrBOZ},`.
  **L1054 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"source", AnyNumeric, Rank::elementalOrBOZ},`。
- **L1055 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mold", SameInt, Rank::scalar}},`.
  **L1055 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mold", SameInt, Rank::scalar}},`。
- **L1056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameInt, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L1056 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameInt, Rank::scalar, IntrinsicClass::transformationalFunction},`。

### Lines 1057-1080

````cpp
    {"transfer",
        {{"source", AnyNumeric, Rank::elementalOrBOZ},
            {"mold", SameReal, Rank::scalar}},
        SameReal, Rank::scalar, IntrinsicClass::transformationalFunction},
    {"transpose", {{"matrix", SameType, Rank::matrix}}, SameType, Rank::matrix,
        IntrinsicClass::transformationalFunction},
    {"trim", {{"string", SameCharNoLen, Rank::scalar}}, SameCharNoLen,
        Rank::scalar, IntrinsicClass::transformationalFunction},
    {"ubound",
        {{"array", AnyData, Rank::arrayOrAssumedRank}, RequiredDIM,
            SizeDefaultKIND},
        KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},
    {"ubound", {{"array", AnyData, Rank::arrayOrAssumedRank}, SizeDefaultKIND},
        KINDInt, Rank::vector, IntrinsicClass::inquiryFunction},
    {"ucobound",
        {{"coarray", AnyData, Rank::coarray}, OptionalDIM, SizeDefaultKIND},
        KINDInt, Rank::scalarIfDim, IntrinsicClass::inquiryFunction},
    {"uint", {{"a", AnyNumeric, Rank::elementalOrBOZ}, DefaultingKIND},
        KINDUnsigned},
    {"umaskl", {{"i", AnyInt}, DefaultingKIND}, KINDUnsigned},
    {"umaskr", {{"i", AnyInt}, DefaultingKIND}, KINDUnsigned},
    {"unlink", {{"path", DefaultChar, Rank::scalar}}, DefaultInt, Rank::scalar,
        IntrinsicClass::transformationalFunction},
    {"unpack",
````
- **L1057 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"transfer",`.
  **L1057 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"transfer",`。
- **L1058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"source", AnyNumeric, Rank::elementalOrBOZ},`.
  **L1058 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"source", AnyNumeric, Rank::elementalOrBOZ},`。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mold", SameReal, Rank::scalar}},`.
  **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mold", SameReal, Rank::scalar}},`。
- **L1060 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameReal, Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L1060 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameReal, Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L1061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"transpose", {{"matrix", SameType, Rank::matrix}}, SameType, Rank::matrix,`.
  **L1061 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"transpose", {{"matrix", SameType, Rank::matrix}}, SameType, Rank::matrix,`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L1063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"trim", {{"string", SameCharNoLen, Rank::scalar}}, SameCharNoLen,`.
  **L1063 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"trim", {{"string", SameCharNoLen, Rank::scalar}}, SameCharNoLen,`。
- **L1064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::transformationalFunction},`.
  **L1064 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::transformationalFunction},`。
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ubound",`.
  **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ubound",`。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"array", AnyData, Rank::arrayOrAssumedRank}, RequiredDIM,`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"array", AnyData, Rank::arrayOrAssumedRank}, RequiredDIM,`。
- **L1067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SizeDefaultKIND},`.
  **L1067 CN**: 继续一个多行参数列表、初始化器或聚合项：`SizeDefaultKIND},`。
- **L1068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`.
  **L1068 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::scalar, IntrinsicClass::inquiryFunction},`。
- **L1069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ubound", {{"array", AnyData, Rank::arrayOrAssumedRank}, SizeDefaultKIND},`.
  **L1069 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ubound", {{"array", AnyData, Rank::arrayOrAssumedRank}, SizeDefaultKIND},`。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::vector, IntrinsicClass::inquiryFunction},`.
  **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::vector, IntrinsicClass::inquiryFunction},`。
- **L1071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ucobound",`.
  **L1071 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ucobound",`。
- **L1072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"coarray", AnyData, Rank::coarray}, OptionalDIM, SizeDefaultKIND},`.
  **L1072 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"coarray", AnyData, Rank::coarray}, OptionalDIM, SizeDefaultKIND},`。
- **L1073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt, Rank::scalarIfDim, IntrinsicClass::inquiryFunction},`.
  **L1073 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt, Rank::scalarIfDim, IntrinsicClass::inquiryFunction},`。
- **L1074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"uint", {{"a", AnyNumeric, Rank::elementalOrBOZ}, DefaultingKIND},`.
  **L1074 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"uint", {{"a", AnyNumeric, Rank::elementalOrBOZ}, DefaultingKIND},`。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDUnsigned},`.
  **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDUnsigned},`。
- **L1076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"umaskl", {{"i", AnyInt}, DefaultingKIND}, KINDUnsigned},`.
  **L1076 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"umaskl", {{"i", AnyInt}, DefaultingKIND}, KINDUnsigned},`。
- **L1077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"umaskr", {{"i", AnyInt}, DefaultingKIND}, KINDUnsigned},`.
  **L1077 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"umaskr", {{"i", AnyInt}, DefaultingKIND}, KINDUnsigned},`。
- **L1078 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"unlink", {{"path", DefaultChar, Rank::scalar}}, DefaultInt, Rank::scalar,`.
  **L1078 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"unlink", {{"path", DefaultChar, Rank::scalar}}, DefaultInt, Rank::scalar,`。
- **L1079 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::transformationalFunction},`.
  **L1079 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::transformationalFunction},`。
- **L1080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"unpack",`.
  **L1080 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"unpack",`。

### Lines 1081-1104

````cpp
        {{"vector", SameType, Rank::vector}, {"mask", AnyLogical, Rank::array},
            {"field", SameType, Rank::conformable}},
        SameType, Rank::conformable, IntrinsicClass::transformationalFunction},
    {"verify",
        {{"string", SameCharNoLen}, {"set", SameCharNoLen},
            {"back", AnyLogical, Rank::elemental, Optionality::optional},
            DefaultingKIND},
        KINDInt},
    {"__builtin_compiler_options", {}, DefaultChar},
    {"__builtin_compiler_version", {}, DefaultChar},
    {"__builtin_fma", {{"f1", SameReal}, {"f2", SameReal}, {"f3", SameReal}},
        SameReal},
    {"__builtin_ieee_int",
        {{"a", AnyFloating}, {"round", IeeeRoundType}, DefaultingKIND},
        KINDInt},
    {"__builtin_ieee_is_nan", {{"a", AnyFloating}}, DefaultLogical},
    {"__builtin_ieee_is_negative", {{"a", AnyFloating}}, DefaultLogical},
    {"__builtin_ieee_is_normal", {{"a", AnyFloating}}, DefaultLogical},
    {"__builtin_ieee_next_after", {{"x", SameReal}, {"y", AnyReal}}, SameReal},
    {"__builtin_ieee_next_down", {{"x", SameReal}}, SameReal},
    {"__builtin_ieee_next_up", {{"x", SameReal}}, SameReal},
    {"__builtin_ieee_real", {{"a", AnyIntOrReal}, DefaultingKIND}, KINDReal},
    {"__builtin_ieee_support_datatype",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
````
- **L1081 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"vector", SameType, Rank::vector}, {"mask", AnyLogical, Rank::array},`.
  **L1081 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"vector", SameType, Rank::vector}, {"mask", AnyLogical, Rank::array},`。
- **L1082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"field", SameType, Rank::conformable}},`.
  **L1082 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"field", SameType, Rank::conformable}},`。
- **L1083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameType, Rank::conformable, IntrinsicClass::transformationalFunction},`.
  **L1083 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameType, Rank::conformable, IntrinsicClass::transformationalFunction},`。
- **L1084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"verify",`.
  **L1084 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"verify",`。
- **L1085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"string", SameCharNoLen}, {"set", SameCharNoLen},`.
  **L1085 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"string", SameCharNoLen}, {"set", SameCharNoLen},`。
- **L1086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::elemental, Optionality::optional},`.
  **L1086 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::elemental, Optionality::optional},`。
- **L1087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultingKIND},`.
  **L1087 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultingKIND},`。
- **L1088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt},`.
  **L1088 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt},`。
- **L1089 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_compiler_options", {}, DefaultChar},`.
  **L1089 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_compiler_options", {}, DefaultChar},`。
- **L1090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_compiler_version", {}, DefaultChar},`.
  **L1090 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_compiler_version", {}, DefaultChar},`。
- **L1091 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_fma", {{"f1", SameReal}, {"f2", SameReal}, {"f3", SameReal}},`.
  **L1091 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_fma", {{"f1", SameReal}, {"f2", SameReal}, {"f3", SameReal}},`。
- **L1092 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SameReal},`.
  **L1092 CN**: 继续一个多行参数列表、初始化器或聚合项：`SameReal},`。
- **L1093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_int",`.
  **L1093 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_int",`。
- **L1094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", AnyFloating}, {"round", IeeeRoundType}, DefaultingKIND},`.
  **L1094 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", AnyFloating}, {"round", IeeeRoundType}, DefaultingKIND},`。
- **L1095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `KINDInt},`.
  **L1095 CN**: 继续一个多行参数列表、初始化器或聚合项：`KINDInt},`。
- **L1096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_is_nan", {{"a", AnyFloating}}, DefaultLogical},`.
  **L1096 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_is_nan", {{"a", AnyFloating}}, DefaultLogical},`。
- **L1097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_is_negative", {{"a", AnyFloating}}, DefaultLogical},`.
  **L1097 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_is_negative", {{"a", AnyFloating}}, DefaultLogical},`。
- **L1098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_is_normal", {{"a", AnyFloating}}, DefaultLogical},`.
  **L1098 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_is_normal", {{"a", AnyFloating}}, DefaultLogical},`。
- **L1099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_next_after", {{"x", SameReal}, {"y", AnyReal}}, SameReal},`.
  **L1099 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_next_after", {{"x", SameReal}, {"y", AnyReal}}, SameReal},`。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_next_down", {{"x", SameReal}}, SameReal},`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_next_down", {{"x", SameReal}}, SameReal},`。
- **L1101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_next_up", {{"x", SameReal}}, SameReal},`.
  **L1101 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_next_up", {{"x", SameReal}}, SameReal},`。
- **L1102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_real", {{"a", AnyIntOrReal}, DefaultingKIND}, KINDReal},`.
  **L1102 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_real", {{"a", AnyIntOrReal}, DefaultingKIND}, KINDReal},`。
- **L1103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_datatype",`.
  **L1103 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_datatype",`。
- **L1104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1104 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。

### Lines 1105-1128

````cpp
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_denormal",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_divide",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_flag",
        {{"flag", IeeeFlagType, Rank::scalar},
            {"x", AnyReal, Rank::known, Optionality::optional,
                common::Intent::In,
                {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_halting", {{"flag", IeeeFlagType, Rank::scalar}},
        DefaultLogical},
    {"__builtin_ieee_support_inf",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_io",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
````
- **L1105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1105 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1106 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_denormal",`.
  **L1107 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_denormal",`。
- **L1108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1108 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。
- **L1109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1109 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1110 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_divide",`.
  **L1111 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_divide",`。
- **L1112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1112 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。
- **L1113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1113 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1114 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_flag",`.
  **L1115 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_flag",`。
- **L1116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"flag", IeeeFlagType, Rank::scalar},`.
  **L1116 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"flag", IeeeFlagType, Rank::scalar},`。
- **L1117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"x", AnyReal, Rank::known, Optionality::optional,`.
  **L1117 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"x", AnyReal, Rank::known, Optionality::optional,`。
- **L1118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L1118 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L1119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1119 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1120 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_halting", {{"flag", IeeeFlagType, Rank::scalar}},`.
  **L1121 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_halting", {{"flag", IeeeFlagType, Rank::scalar}},`。
- **L1122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1122 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_inf",`.
  **L1123 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_inf",`。
- **L1124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1124 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。
- **L1125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1125 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1126 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_io",`.
  **L1127 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_io",`。
- **L1128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1128 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。

### Lines 1129-1152

````cpp
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_nan",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_rounding",
        {{"round_value", IeeeRoundType, Rank::scalar},
            {"x", AnyReal, Rank::known, Optionality::optional,
                common::Intent::In,
                {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_sqrt",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_standard",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_ieee_support_subnormal",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
````
- **L1129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1129 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1130 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_nan",`.
  **L1131 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_nan",`。
- **L1132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1132 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_rounding",`.
  **L1135 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_rounding",`。
- **L1136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"round_value", IeeeRoundType, Rank::scalar},`.
  **L1136 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"round_value", IeeeRoundType, Rank::scalar},`。
- **L1137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"x", AnyReal, Rank::known, Optionality::optional,`.
  **L1137 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"x", AnyReal, Rank::known, Optionality::optional,`。
- **L1138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In,`.
  **L1138 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In,`。
- **L1139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1139 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1140 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_sqrt",`.
  **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_sqrt",`。
- **L1142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1142 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。
- **L1143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1143 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1144 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_standard",`.
  **L1145 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_standard",`。
- **L1146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1146 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。
- **L1147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1147 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1148 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_subnormal",`.
  **L1149 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_subnormal",`。
- **L1150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1150 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。
- **L1151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1151 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1152 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。

### Lines 1153-1176

````cpp
    {"__builtin_ieee_support_underflow_control",
        {{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,
            {ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},
        DefaultLogical},
    {"__builtin_numeric_storage_size", {}, DefaultInt},
};

// TODO: Non-standard intrinsic functions
//  SHIFT,
//  COMPL, EQV, NEQV, INT8, JINT, JNINT, KNINT,
//  QCMPLX, QEXT, QFLOAT, QREAL, DNUM,
//  INUM, JNUM, KNUM, QNUM, RNUM, RAN, RANF, ILEN,
//  MCLOCK, SECNDS, COTAN, IBCHNG, ISHA, ISHC, ISHL, IXOR
//  IARG, IARGC, NARGS, NUMARG, BADDRESS, IADDR, CACHESIZE,
//  EOF, FP_CLASS, INT_PTR_KIND, MALLOC
//  probably more (these are PGI + Intel, possibly incomplete)
// TODO: Optionally warn on use of non-standard intrinsics:
//  LOC, probably others
// TODO: Optionally warn on operand promotion extension

// Aliases for a few generic procedures for legacy compatibility and builtins.
static const std::pair<const char *, const char *> genericAlias[]{
    {"and", "iand"},
    {"getenv", "get_environment_variable"},
````
- **L1153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_support_underflow_control",`.
  **L1153 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_support_underflow_control",`。
- **L1154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`.
  **L1154 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyReal, Rank::known, Optionality::optional, common::Intent::In,`。
- **L1155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`.
  **L1155 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::canBeMoldNull, ArgFlag::onlyConstantInquiry}}},`。
- **L1156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1156 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_numeric_storage_size", {}, DefaultInt},`.
  **L1157 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_numeric_storage_size", {}, DefaultInt},`。
- **L1158 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1158 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1160 EN**: Comment records a pending task or caution: `TODO: Non-standard intrinsic functions`.
  **L1160 CN**: 注释记录待办事项或注意点：`TODO: Non-standard intrinsic functions`。
- **L1161 EN**: Comment explains nearby logic, intent, or metadata: `SHIFT,`.
  **L1161 CN**: 注释说明附近代码的逻辑、意图或元数据：`SHIFT,`。
- **L1162 EN**: Comment explains nearby logic, intent, or metadata: `COMPL, EQV, NEQV, INT8, JINT, JNINT, KNINT,`.
  **L1162 CN**: 注释说明附近代码的逻辑、意图或元数据：`COMPL, EQV, NEQV, INT8, JINT, JNINT, KNINT,`。
- **L1163 EN**: Comment explains nearby logic, intent, or metadata: `QCMPLX, QEXT, QFLOAT, QREAL, DNUM,`.
  **L1163 CN**: 注释说明附近代码的逻辑、意图或元数据：`QCMPLX, QEXT, QFLOAT, QREAL, DNUM,`。
- **L1164 EN**: Comment explains nearby logic, intent, or metadata: `INUM, JNUM, KNUM, QNUM, RNUM, RAN, RANF, ILEN,`.
  **L1164 CN**: 注释说明附近代码的逻辑、意图或元数据：`INUM, JNUM, KNUM, QNUM, RNUM, RAN, RANF, ILEN,`。
- **L1165 EN**: Comment explains nearby logic, intent, or metadata: `MCLOCK, SECNDS, COTAN, IBCHNG, ISHA, ISHC, ISHL, IXOR`.
  **L1165 CN**: 注释说明附近代码的逻辑、意图或元数据：`MCLOCK, SECNDS, COTAN, IBCHNG, ISHA, ISHC, ISHL, IXOR`。
- **L1166 EN**: Comment explains nearby logic, intent, or metadata: `IARG, IARGC, NARGS, NUMARG, BADDRESS, IADDR, CACHESIZE,`.
  **L1166 CN**: 注释说明附近代码的逻辑、意图或元数据：`IARG, IARGC, NARGS, NUMARG, BADDRESS, IADDR, CACHESIZE,`。
- **L1167 EN**: Comment explains nearby logic, intent, or metadata: `EOF, FP_CLASS, INT_PTR_KIND, MALLOC`.
  **L1167 CN**: 注释说明附近代码的逻辑、意图或元数据：`EOF, FP_CLASS, INT_PTR_KIND, MALLOC`。
- **L1168 EN**: Comment explains nearby logic, intent, or metadata: `probably more (these are PGI + Intel, possibly incomplete)`.
  **L1168 CN**: 注释说明附近代码的逻辑、意图或元数据：`probably more (these are PGI + Intel, possibly incomplete)`。
- **L1169 EN**: Comment records a pending task or caution: `TODO: Optionally warn on use of non-standard intrinsics:`.
  **L1169 CN**: 注释记录待办事项或注意点：`TODO: Optionally warn on use of non-standard intrinsics:`。
- **L1170 EN**: Comment explains nearby logic, intent, or metadata: `LOC, probably others`.
  **L1170 CN**: 注释说明附近代码的逻辑、意图或元数据：`LOC, probably others`。
- **L1171 EN**: Comment records a pending task or caution: `TODO: Optionally warn on operand promotion extension`.
  **L1171 CN**: 注释记录待办事项或注意点：`TODO: Optionally warn on operand promotion extension`。
- **L1172 EN**: Blank line separating nearby declarations or logic blocks.
  **L1172 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1173 EN**: Comment explains nearby logic, intent, or metadata: `Aliases for a few generic procedures for legacy compatibility and builtins.`.
  **L1173 CN**: 注释说明附近代码的逻辑、意图或元数据：`Aliases for a few generic procedures for legacy compatibility and builtins.`。
- **L1174 EN**: Continues the surrounding expression or declaration: `static const std::pair<const char *, const char *> genericAlias[]{`.
  **L1174 CN**: 继续构造周围的表达式或声明：`static const std::pair<const char *, const char *> genericAlias[]{`。
- **L1175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"and", "iand"},`.
  **L1175 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"and", "iand"},`。
- **L1176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"getenv", "get_environment_variable"},`.
  **L1176 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"getenv", "get_environment_variable"},`。

### Lines 1177-1200

````cpp
    {"fseek64", "fseek"},
    {"fseeko64", "fseek"}, // SUN
    {"fseeki8", "fseek"}, // Intel
    {"ftell64", "ftell"},
    {"ftello64", "ftell"}, // SUN
    {"ftelli8", "ftell"}, // Intel
    {"imag", "aimag"},
    {"lshift", "shiftl"},
    {"or", "ior"},
    {"rshift", "shifta"},
    {"unsigned", "uint"}, // Sun vs gfortran names
    {"xor", "ieor"},
    {"__builtin_ieee_selected_real_kind", "selected_real_kind"},
    {IntrinsicProcTable::BuiltinIntName, "int"},
};

// The following table contains the intrinsic functions listed in
// Tables 16.2 and 16.3 in Fortran 2018.  The "unrestricted" functions
// in Table 16.2 can be used as actual arguments, PROCEDURE() interfaces,
// and procedure pointer targets.
// Note that the restricted conversion functions dcmplx, dreal, float, idint,
// ifix, and sngl are extended to accept any argument kind because this is a
// common Fortran compilers behavior, and as far as we can tell, is safe and
// useful.
````
- **L1177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"fseek64", "fseek"},`.
  **L1177 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"fseek64", "fseek"},`。
- **L1178 EN**: Continues the surrounding expression or declaration: `{"fseeko64", "fseek"}, // SUN`.
  **L1178 CN**: 继续构造周围的表达式或声明：`{"fseeko64", "fseek"}, // SUN`。
- **L1179 EN**: Continues the surrounding expression or declaration: `{"fseeki8", "fseek"}, // Intel`.
  **L1179 CN**: 继续构造周围的表达式或声明：`{"fseeki8", "fseek"}, // Intel`。
- **L1180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ftell64", "ftell"},`.
  **L1180 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ftell64", "ftell"},`。
- **L1181 EN**: Continues the surrounding expression or declaration: `{"ftello64", "ftell"}, // SUN`.
  **L1181 CN**: 继续构造周围的表达式或声明：`{"ftello64", "ftell"}, // SUN`。
- **L1182 EN**: Continues the surrounding expression or declaration: `{"ftelli8", "ftell"}, // Intel`.
  **L1182 CN**: 继续构造周围的表达式或声明：`{"ftelli8", "ftell"}, // Intel`。
- **L1183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"imag", "aimag"},`.
  **L1183 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"imag", "aimag"},`。
- **L1184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"lshift", "shiftl"},`.
  **L1184 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"lshift", "shiftl"},`。
- **L1185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"or", "ior"},`.
  **L1185 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"or", "ior"},`。
- **L1186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"rshift", "shifta"},`.
  **L1186 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"rshift", "shifta"},`。
- **L1187 EN**: Continues the surrounding expression or declaration: `{"unsigned", "uint"}, // Sun vs gfortran names`.
  **L1187 CN**: 继续构造周围的表达式或声明：`{"unsigned", "uint"}, // Sun vs gfortran names`。
- **L1188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xor", "ieor"},`.
  **L1188 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xor", "ieor"},`。
- **L1189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_ieee_selected_real_kind", "selected_real_kind"},`.
  **L1189 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_ieee_selected_real_kind", "selected_real_kind"},`。
- **L1190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IntrinsicProcTable::BuiltinIntName, "int"},`.
  **L1190 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IntrinsicProcTable::BuiltinIntName, "int"},`。
- **L1191 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1191 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1192 EN**: Blank line separating nearby declarations or logic blocks.
  **L1192 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1193 EN**: Comment explains nearby logic, intent, or metadata: `The following table contains the intrinsic functions listed in`.
  **L1193 CN**: 注释说明附近代码的逻辑、意图或元数据：`The following table contains the intrinsic functions listed in`。
- **L1194 EN**: Comment explains nearby logic, intent, or metadata: `Tables 16.2 and 16.3 in Fortran 2018.  The "unrestricted" functions`.
  **L1194 CN**: 注释说明附近代码的逻辑、意图或元数据：`Tables 16.2 and 16.3 in Fortran 2018.  The "unrestricted" functions`。
- **L1195 EN**: Comment explains nearby logic, intent, or metadata: `in Table 16.2 can be used as actual arguments, PROCEDURE() interfaces,`.
  **L1195 CN**: 注释说明附近代码的逻辑、意图或元数据：`in Table 16.2 can be used as actual arguments, PROCEDURE() interfaces,`。
- **L1196 EN**: Comment explains nearby logic, intent, or metadata: `and procedure pointer targets.`.
  **L1196 CN**: 注释说明附近代码的逻辑、意图或元数据：`and procedure pointer targets.`。
- **L1197 EN**: Comment explains nearby logic, intent, or metadata: `Note that the restricted conversion functions dcmplx, dreal, float, idint,`.
  **L1197 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that the restricted conversion functions dcmplx, dreal, float, idint,`。
- **L1198 EN**: Comment explains nearby logic, intent, or metadata: `ifix, and sngl are extended to accept any argument kind because this is a`.
  **L1198 CN**: 注释说明附近代码的逻辑、意图或元数据：`ifix, and sngl are extended to accept any argument kind because this is a`。
- **L1199 EN**: Comment explains nearby logic, intent, or metadata: `common Fortran compilers behavior, and as far as we can tell, is safe and`.
  **L1199 CN**: 注释说明附近代码的逻辑、意图或元数据：`common Fortran compilers behavior, and as far as we can tell, is safe and`。
- **L1200 EN**: Comment explains nearby logic, intent, or metadata: `useful.`.
  **L1200 CN**: 注释说明附近代码的逻辑、意图或元数据：`useful.`。

### Lines 1201-1224

````cpp
struct SpecificIntrinsicInterface : public IntrinsicInterface {
  const char *generic{nullptr};
  bool isRestrictedSpecific{false};
  // Exact actual/dummy type matching is required by default for specific
  // intrinsics. If useGenericAndForceResultType is set, then the probing will
  // also attempt to use the related generic intrinsic and to convert the result
  // to the specific intrinsic result type if needed. This also prevents
  // using the generic name so that folding can insert the conversion on the
  // result and not the arguments.
  //
  // This is not enabled on all specific intrinsics because an alternative
  // is to convert the actual arguments to the required dummy types and this is
  // not numerically equivalent.
  //  e.g. IABS(INT(i, 4)) not equiv to INT(ABS(i), 4).
  // This is allowed for restricted min/max specific functions because
  // the expected behavior is clear from their definitions. A warning is though
  // always emitted because other compilers' behavior is not ubiquitous here and
  // the results in case of conversion overflow might not be equivalent.
  // e.g for MIN0: INT(MIN(2147483647_8, 2*2147483647_8), 4) = 2147483647_4
  // but: MIN(INT(2147483647_8, 4), INT(2*2147483647_8, 4)) = -2_4
  // xlf and ifort return the first, and pgfortran the later. f18 will return
  // the first because this matches more closely the MIN0 definition in
  // Fortran 2018 table 16.3 (although it is still an extension to allow
  // non default integer argument in MIN0).
````
- **L1201 EN**: Declares struct `SpecificIntrinsicInterface`.
  **L1201 CN**: 声明 struct `SpecificIntrinsicInterface`。
- **L1202 EN**: Executes a standalone statement or declaration: `const char *generic{nullptr};`.
  **L1202 CN**: 执行一条独立语句或声明：`const char *generic{nullptr};`。
- **L1203 EN**: Executes a standalone statement or declaration: `bool isRestrictedSpecific{false};`.
  **L1203 CN**: 执行一条独立语句或声明：`bool isRestrictedSpecific{false};`。
- **L1204 EN**: Comment explains nearby logic, intent, or metadata: `Exact actual/dummy type matching is required by default for specific`.
  **L1204 CN**: 注释说明附近代码的逻辑、意图或元数据：`Exact actual/dummy type matching is required by default for specific`。
- **L1205 EN**: Comment explains nearby logic, intent, or metadata: `intrinsics. If useGenericAndForceResultType is set, then the probing will`.
  **L1205 CN**: 注释说明附近代码的逻辑、意图或元数据：`intrinsics. If useGenericAndForceResultType is set, then the probing will`。
- **L1206 EN**: Comment explains nearby logic, intent, or metadata: `also attempt to use the related generic intrinsic and to convert the result`.
  **L1206 CN**: 注释说明附近代码的逻辑、意图或元数据：`also attempt to use the related generic intrinsic and to convert the result`。
- **L1207 EN**: Comment explains nearby logic, intent, or metadata: `to the specific intrinsic result type if needed. This also prevents`.
  **L1207 CN**: 注释说明附近代码的逻辑、意图或元数据：`to the specific intrinsic result type if needed. This also prevents`。
- **L1208 EN**: Comment explains nearby logic, intent, or metadata: `using the generic name so that folding can insert the conversion on the`.
  **L1208 CN**: 注释说明附近代码的逻辑、意图或元数据：`using the generic name so that folding can insert the conversion on the`。
- **L1209 EN**: Comment explains nearby logic, intent, or metadata: `result and not the arguments.`.
  **L1209 CN**: 注释说明附近代码的逻辑、意图或元数据：`result and not the arguments.`。
- **L1210 EN**: Separator comment used for visual grouping.
  **L1210 CN**: 用于视觉分组的分隔注释。
- **L1211 EN**: Comment explains nearby logic, intent, or metadata: `This is not enabled on all specific intrinsics because an alternative`.
  **L1211 CN**: 注释说明附近代码的逻辑、意图或元数据：`This is not enabled on all specific intrinsics because an alternative`。
- **L1212 EN**: Comment explains nearby logic, intent, or metadata: `is to convert the actual arguments to the required dummy types and this is`.
  **L1212 CN**: 注释说明附近代码的逻辑、意图或元数据：`is to convert the actual arguments to the required dummy types and this is`。
- **L1213 EN**: Comment explains nearby logic, intent, or metadata: `not numerically equivalent.`.
  **L1213 CN**: 注释说明附近代码的逻辑、意图或元数据：`not numerically equivalent.`。
- **L1214 EN**: Comment explains nearby logic, intent, or metadata: `e.g. IABS(INT(i, 4)) not equiv to INT(ABS(i), 4).`.
  **L1214 CN**: 注释说明附近代码的逻辑、意图或元数据：`e.g. IABS(INT(i, 4)) not equiv to INT(ABS(i), 4).`。
- **L1215 EN**: Comment explains nearby logic, intent, or metadata: `This is allowed for restricted min/max specific functions because`.
  **L1215 CN**: 注释说明附近代码的逻辑、意图或元数据：`This is allowed for restricted min/max specific functions because`。
- **L1216 EN**: Comment explains nearby logic, intent, or metadata: `the expected behavior is clear from their definitions. A warning is though`.
  **L1216 CN**: 注释说明附近代码的逻辑、意图或元数据：`the expected behavior is clear from their definitions. A warning is though`。
- **L1217 EN**: Comment explains nearby logic, intent, or metadata: `always emitted because other compilers' behavior is not ubiquitous here and`.
  **L1217 CN**: 注释说明附近代码的逻辑、意图或元数据：`always emitted because other compilers' behavior is not ubiquitous here and`。
- **L1218 EN**: Comment explains nearby logic, intent, or metadata: `the results in case of conversion overflow might not be equivalent.`.
  **L1218 CN**: 注释说明附近代码的逻辑、意图或元数据：`the results in case of conversion overflow might not be equivalent.`。
- **L1219 EN**: Comment explains nearby logic, intent, or metadata: `e.g for MIN0: INT(MIN(2147483647_8, 2*2147483647_8), 4) = 2147483647_4`.
  **L1219 CN**: 注释说明附近代码的逻辑、意图或元数据：`e.g for MIN0: INT(MIN(2147483647_8, 2*2147483647_8), 4) = 2147483647_4`。
- **L1220 EN**: Comment explains nearby logic, intent, or metadata: `but: MIN(INT(2147483647_8, 4), INT(2*2147483647_8, 4)) = -2_4`.
  **L1220 CN**: 注释说明附近代码的逻辑、意图或元数据：`but: MIN(INT(2147483647_8, 4), INT(2*2147483647_8, 4)) = -2_4`。
- **L1221 EN**: Comment explains nearby logic, intent, or metadata: `xlf and ifort return the first, and pgfortran the later. f18 will return`.
  **L1221 CN**: 注释说明附近代码的逻辑、意图或元数据：`xlf and ifort return the first, and pgfortran the later. f18 will return`。
- **L1222 EN**: Comment explains nearby logic, intent, or metadata: `the first because this matches more closely the MIN0 definition in`.
  **L1222 CN**: 注释说明附近代码的逻辑、意图或元数据：`the first because this matches more closely the MIN0 definition in`。
- **L1223 EN**: Comment explains nearby logic, intent, or metadata: `Fortran 2018 table 16.3 (although it is still an extension to allow`.
  **L1223 CN**: 注释说明附近代码的逻辑、意图或元数据：`Fortran 2018 table 16.3 (although it is still an extension to allow`。
- **L1224 EN**: Comment explains nearby logic, intent, or metadata: `non default integer argument in MIN0).`.
  **L1224 CN**: 注释说明附近代码的逻辑、意图或元数据：`non default integer argument in MIN0).`。

### Lines 1225-1248

````cpp
  bool useGenericAndForceResultType{false};
};

static const SpecificIntrinsicInterface specificIntrinsicFunction[]{
    {{"abs", {{"a", DefaultReal}}, DefaultReal}},
    {{"acos", {{"x", DefaultReal}}, DefaultReal}},
    {{"aimag", {{"z", DefaultComplex}}, DefaultReal}},
    {{"aint", {{"a", DefaultReal}}, DefaultReal}},
    {{"alog", {{"x", DefaultReal}}, DefaultReal}, "log"},
    {{"alog10", {{"x", DefaultReal}}, DefaultReal}, "log10"},
    {{"amax0",
         {{"a1", DefaultInt}, {"a2", DefaultInt},
             {"a3", DefaultInt, Rank::elemental, Optionality::repeats}},
         DefaultReal},
        "max", true, true},
    {{"amax1",
         {{"a1", DefaultReal}, {"a2", DefaultReal},
             {"a3", DefaultReal, Rank::elemental, Optionality::repeats}},
         DefaultReal},
        "max", true, true},
    {{"amin0",
         {{"a1", DefaultInt}, {"a2", DefaultInt},
             {"a3", DefaultInt, Rank::elemental, Optionality::repeats}},
         DefaultReal},
````
- **L1225 EN**: Executes a standalone statement or declaration: `bool useGenericAndForceResultType{false};`.
  **L1225 CN**: 执行一条独立语句或声明：`bool useGenericAndForceResultType{false};`。
- **L1226 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1226 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1227 EN**: Blank line separating nearby declarations or logic blocks.
  **L1227 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1228 EN**: Continues the surrounding expression or declaration: `static const SpecificIntrinsicInterface specificIntrinsicFunction[]{`.
  **L1228 CN**: 继续构造周围的表达式或声明：`static const SpecificIntrinsicInterface specificIntrinsicFunction[]{`。
- **L1229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"abs", {{"a", DefaultReal}}, DefaultReal}},`.
  **L1229 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"abs", {{"a", DefaultReal}}, DefaultReal}},`。
- **L1230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"acos", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1230 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"acos", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"aimag", {{"z", DefaultComplex}}, DefaultReal}},`.
  **L1231 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"aimag", {{"z", DefaultComplex}}, DefaultReal}},`。
- **L1232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"aint", {{"a", DefaultReal}}, DefaultReal}},`.
  **L1232 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"aint", {{"a", DefaultReal}}, DefaultReal}},`。
- **L1233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"alog", {{"x", DefaultReal}}, DefaultReal}, "log"},`.
  **L1233 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"alog", {{"x", DefaultReal}}, DefaultReal}, "log"},`。
- **L1234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"alog10", {{"x", DefaultReal}}, DefaultReal}, "log10"},`.
  **L1234 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"alog10", {{"x", DefaultReal}}, DefaultReal}, "log10"},`。
- **L1235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"amax0",`.
  **L1235 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"amax0",`。
- **L1236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DefaultInt}, {"a2", DefaultInt},`.
  **L1236 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DefaultInt}, {"a2", DefaultInt},`。
- **L1237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DefaultInt, Rank::elemental, Optionality::repeats}},`.
  **L1237 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DefaultInt, Rank::elemental, Optionality::repeats}},`。
- **L1238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultReal},`.
  **L1238 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultReal},`。
- **L1239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"max", true, true},`.
  **L1239 CN**: 继续一个多行参数列表、初始化器或聚合项：`"max", true, true},`。
- **L1240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"amax1",`.
  **L1240 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"amax1",`。
- **L1241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DefaultReal}, {"a2", DefaultReal},`.
  **L1241 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DefaultReal}, {"a2", DefaultReal},`。
- **L1242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DefaultReal, Rank::elemental, Optionality::repeats}},`.
  **L1242 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DefaultReal, Rank::elemental, Optionality::repeats}},`。
- **L1243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultReal},`.
  **L1243 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultReal},`。
- **L1244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"max", true, true},`.
  **L1244 CN**: 继续一个多行参数列表、初始化器或聚合项：`"max", true, true},`。
- **L1245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"amin0",`.
  **L1245 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"amin0",`。
- **L1246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DefaultInt}, {"a2", DefaultInt},`.
  **L1246 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DefaultInt}, {"a2", DefaultInt},`。
- **L1247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DefaultInt, Rank::elemental, Optionality::repeats}},`.
  **L1247 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DefaultInt, Rank::elemental, Optionality::repeats}},`。
- **L1248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultReal},`.
  **L1248 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultReal},`。

### Lines 1249-1272

````cpp
        "min", true, true},
    {{"amin1",
         {{"a1", DefaultReal}, {"a2", DefaultReal},
             {"a3", DefaultReal, Rank::elemental, Optionality::repeats}},
         DefaultReal},
        "min", true, true},
    {{"amod", {{"a", DefaultReal}, {"p", DefaultReal}}, DefaultReal}, "mod"},
    {{"anint", {{"a", DefaultReal}}, DefaultReal}},
    {{"asin", {{"x", DefaultReal}}, DefaultReal}},
    {{"atan", {{"x", DefaultReal}}, DefaultReal}},
    {{"atan2", {{"y", DefaultReal}, {"x", DefaultReal}}, DefaultReal}},
    {{"babs", {{"a", TypePattern{IntType, KindCode::exactKind, 1}}},
         TypePattern{IntType, KindCode::exactKind, 1}},
        "abs"},
    {{"cabs", {{"a", DefaultComplex}}, DefaultReal}, "abs"},
    {{"ccos", {{"x", DefaultComplex}}, DefaultComplex}, "cos"},
    {{"cdabs", {{"a", DoublePrecisionComplex}}, DoublePrecision}, "abs"},
    {{"cdcos", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "cos"},
    {{"cdexp", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "exp"},
    {{"cdlog", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "log"},
    {{"cdsin", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "sin"},
    {{"cdsqrt", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex},
        "sqrt"},
    {{"cexp", {{"x", DefaultComplex}}, DefaultComplex}, "exp"},
````
- **L1249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"min", true, true},`.
  **L1249 CN**: 继续一个多行参数列表、初始化器或聚合项：`"min", true, true},`。
- **L1250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"amin1",`.
  **L1250 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"amin1",`。
- **L1251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DefaultReal}, {"a2", DefaultReal},`.
  **L1251 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DefaultReal}, {"a2", DefaultReal},`。
- **L1252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DefaultReal, Rank::elemental, Optionality::repeats}},`.
  **L1252 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DefaultReal, Rank::elemental, Optionality::repeats}},`。
- **L1253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultReal},`.
  **L1253 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultReal},`。
- **L1254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"min", true, true},`.
  **L1254 CN**: 继续一个多行参数列表、初始化器或聚合项：`"min", true, true},`。
- **L1255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"amod", {{"a", DefaultReal}, {"p", DefaultReal}}, DefaultReal}, "mod"},`.
  **L1255 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"amod", {{"a", DefaultReal}, {"p", DefaultReal}}, DefaultReal}, "mod"},`。
- **L1256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"anint", {{"a", DefaultReal}}, DefaultReal}},`.
  **L1256 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"anint", {{"a", DefaultReal}}, DefaultReal}},`。
- **L1257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"asin", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1257 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"asin", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atan", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1258 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atan", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atan2", {{"y", DefaultReal}, {"x", DefaultReal}}, DefaultReal}},`.
  **L1259 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atan2", {{"y", DefaultReal}, {"x", DefaultReal}}, DefaultReal}},`。
- **L1260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"babs", {{"a", TypePattern{IntType, KindCode::exactKind, 1}}},`.
  **L1260 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"babs", {{"a", TypePattern{IntType, KindCode::exactKind, 1}}},`。
- **L1261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 1}},`.
  **L1261 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 1}},`。
- **L1262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"abs"},`.
  **L1262 CN**: 继续一个多行参数列表、初始化器或聚合项：`"abs"},`。
- **L1263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cabs", {{"a", DefaultComplex}}, DefaultReal}, "abs"},`.
  **L1263 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cabs", {{"a", DefaultComplex}}, DefaultReal}, "abs"},`。
- **L1264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"ccos", {{"x", DefaultComplex}}, DefaultComplex}, "cos"},`.
  **L1264 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"ccos", {{"x", DefaultComplex}}, DefaultComplex}, "cos"},`。
- **L1265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cdabs", {{"a", DoublePrecisionComplex}}, DoublePrecision}, "abs"},`.
  **L1265 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cdabs", {{"a", DoublePrecisionComplex}}, DoublePrecision}, "abs"},`。
- **L1266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cdcos", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "cos"},`.
  **L1266 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cdcos", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "cos"},`。
- **L1267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cdexp", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "exp"},`.
  **L1267 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cdexp", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "exp"},`。
- **L1268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cdlog", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "log"},`.
  **L1268 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cdlog", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "log"},`。
- **L1269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cdsin", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "sin"},`.
  **L1269 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cdsin", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex}, "sin"},`。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cdsqrt", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex},`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cdsqrt", {{"x", DoublePrecisionComplex}}, DoublePrecisionComplex},`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"sqrt"},`.
  **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`"sqrt"},`。
- **L1272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cexp", {{"x", DefaultComplex}}, DefaultComplex}, "exp"},`.
  **L1272 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cexp", {{"x", DefaultComplex}}, DefaultComplex}, "exp"},`。

### Lines 1273-1296

````cpp
    {{"clog", {{"x", DefaultComplex}}, DefaultComplex}, "log"},
    {{"conjg", {{"z", DefaultComplex}}, DefaultComplex}},
    {{"cos", {{"x", DefaultReal}}, DefaultReal}},
    {{"cosh", {{"x", DefaultReal}}, DefaultReal}},
    {{"csin", {{"x", DefaultComplex}}, DefaultComplex}, "sin"},
    {{"csqrt", {{"x", DefaultComplex}}, DefaultComplex}, "sqrt"},
    {{"ctan", {{"x", DefaultComplex}}, DefaultComplex}, "tan"},
    {{"dabs", {{"a", DoublePrecision}}, DoublePrecision}, "abs"},
    {{"dacos", {{"x", DoublePrecision}}, DoublePrecision}, "acos"},
    {{"dasin", {{"x", DoublePrecision}}, DoublePrecision}, "asin"},
    {{"datan", {{"x", DoublePrecision}}, DoublePrecision}, "atan"},
    {{"datan2", {{"y", DoublePrecision}, {"x", DoublePrecision}},
         DoublePrecision},
        "atan2"},
    {{"dcmplx", {{"x", AnyComplex}}, DoublePrecisionComplex}, "cmplx", true},
    {{"dcmplx",
         {{"x", AnyIntOrReal, Rank::elementalOrBOZ},
             {"y", AnyIntOrReal, Rank::elementalOrBOZ, Optionality::optional}},
         DoublePrecisionComplex},
        "cmplx", true},
    {{"dconjg", {{"z", DoublePrecisionComplex}}, DoublePrecisionComplex},
        "conjg"},
    {{"dcos", {{"x", DoublePrecision}}, DoublePrecision}, "cos"},
    {{"dcosh", {{"x", DoublePrecision}}, DoublePrecision}, "cosh"},
````
- **L1273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"clog", {{"x", DefaultComplex}}, DefaultComplex}, "log"},`.
  **L1273 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"clog", {{"x", DefaultComplex}}, DefaultComplex}, "log"},`。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"conjg", {{"z", DefaultComplex}}, DefaultComplex}},`.
  **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"conjg", {{"z", DefaultComplex}}, DefaultComplex}},`。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cos", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cos", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"cosh", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1276 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"cosh", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"csin", {{"x", DefaultComplex}}, DefaultComplex}, "sin"},`.
  **L1277 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"csin", {{"x", DefaultComplex}}, DefaultComplex}, "sin"},`。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"csqrt", {{"x", DefaultComplex}}, DefaultComplex}, "sqrt"},`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"csqrt", {{"x", DefaultComplex}}, DefaultComplex}, "sqrt"},`。
- **L1279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"ctan", {{"x", DefaultComplex}}, DefaultComplex}, "tan"},`.
  **L1279 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"ctan", {{"x", DefaultComplex}}, DefaultComplex}, "tan"},`。
- **L1280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dabs", {{"a", DoublePrecision}}, DoublePrecision}, "abs"},`.
  **L1280 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dabs", {{"a", DoublePrecision}}, DoublePrecision}, "abs"},`。
- **L1281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dacos", {{"x", DoublePrecision}}, DoublePrecision}, "acos"},`.
  **L1281 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dacos", {{"x", DoublePrecision}}, DoublePrecision}, "acos"},`。
- **L1282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dasin", {{"x", DoublePrecision}}, DoublePrecision}, "asin"},`.
  **L1282 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dasin", {{"x", DoublePrecision}}, DoublePrecision}, "asin"},`。
- **L1283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"datan", {{"x", DoublePrecision}}, DoublePrecision}, "atan"},`.
  **L1283 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"datan", {{"x", DoublePrecision}}, DoublePrecision}, "atan"},`。
- **L1284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"datan2", {{"y", DoublePrecision}, {"x", DoublePrecision}},`.
  **L1284 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"datan2", {{"y", DoublePrecision}, {"x", DoublePrecision}},`。
- **L1285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DoublePrecision},`.
  **L1285 CN**: 继续一个多行参数列表、初始化器或聚合项：`DoublePrecision},`。
- **L1286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"atan2"},`.
  **L1286 CN**: 继续一个多行参数列表、初始化器或聚合项：`"atan2"},`。
- **L1287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dcmplx", {{"x", AnyComplex}}, DoublePrecisionComplex}, "cmplx", true},`.
  **L1287 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dcmplx", {{"x", AnyComplex}}, DoublePrecisionComplex}, "cmplx", true},`。
- **L1288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dcmplx",`.
  **L1288 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dcmplx",`。
- **L1289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"x", AnyIntOrReal, Rank::elementalOrBOZ},`.
  **L1289 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"x", AnyIntOrReal, Rank::elementalOrBOZ},`。
- **L1290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"y", AnyIntOrReal, Rank::elementalOrBOZ, Optionality::optional}},`.
  **L1290 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"y", AnyIntOrReal, Rank::elementalOrBOZ, Optionality::optional}},`。
- **L1291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DoublePrecisionComplex},`.
  **L1291 CN**: 继续一个多行参数列表、初始化器或聚合项：`DoublePrecisionComplex},`。
- **L1292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"cmplx", true},`.
  **L1292 CN**: 继续一个多行参数列表、初始化器或聚合项：`"cmplx", true},`。
- **L1293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dconjg", {{"z", DoublePrecisionComplex}}, DoublePrecisionComplex},`.
  **L1293 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dconjg", {{"z", DoublePrecisionComplex}}, DoublePrecisionComplex},`。
- **L1294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"conjg"},`.
  **L1294 CN**: 继续一个多行参数列表、初始化器或聚合项：`"conjg"},`。
- **L1295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dcos", {{"x", DoublePrecision}}, DoublePrecision}, "cos"},`.
  **L1295 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dcos", {{"x", DoublePrecision}}, DoublePrecision}, "cos"},`。
- **L1296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dcosh", {{"x", DoublePrecision}}, DoublePrecision}, "cosh"},`.
  **L1296 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dcosh", {{"x", DoublePrecision}}, DoublePrecision}, "cosh"},`。

### Lines 1297-1320

````cpp
    {{"ddim", {{"x", DoublePrecision}, {"y", DoublePrecision}},
         DoublePrecision},
        "dim"},
    {{"derf", {{"x", DoublePrecision}}, DoublePrecision}, "erf"},
    {{"derfc", {{"x", DoublePrecision}}, DoublePrecision}, "erfc"},
    {{"derfc_scaled", {{"x", DoublePrecision}}, DoublePrecision},
        "erfc_scaled"},
    {{"dexp", {{"x", DoublePrecision}}, DoublePrecision}, "exp"},
    {{"dfloat", {{"a", AnyInt}}, DoublePrecision}, "real", true},
    {{"dim", {{"x", DefaultReal}, {"y", DefaultReal}}, DefaultReal}},
    {{"dimag", {{"z", DoublePrecisionComplex}}, DoublePrecision}, "aimag"},
    {{"dint", {{"a", DoublePrecision}}, DoublePrecision}, "aint"},
    {{"dlog", {{"x", DoublePrecision}}, DoublePrecision}, "log"},
    {{"dlog10", {{"x", DoublePrecision}}, DoublePrecision}, "log10"},
    {{"dmax1",
         {{"a1", DoublePrecision}, {"a2", DoublePrecision},
             {"a3", DoublePrecision, Rank::elemental, Optionality::repeats}},
         DoublePrecision},
        "max", true, true},
    {{"dmin1",
         {{"a1", DoublePrecision}, {"a2", DoublePrecision},
             {"a3", DoublePrecision, Rank::elemental, Optionality::repeats}},
         DoublePrecision},
        "min", true, true},
````
- **L1297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"ddim", {{"x", DoublePrecision}, {"y", DoublePrecision}},`.
  **L1297 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"ddim", {{"x", DoublePrecision}, {"y", DoublePrecision}},`。
- **L1298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DoublePrecision},`.
  **L1298 CN**: 继续一个多行参数列表、初始化器或聚合项：`DoublePrecision},`。
- **L1299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"dim"},`.
  **L1299 CN**: 继续一个多行参数列表、初始化器或聚合项：`"dim"},`。
- **L1300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"derf", {{"x", DoublePrecision}}, DoublePrecision}, "erf"},`.
  **L1300 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"derf", {{"x", DoublePrecision}}, DoublePrecision}, "erf"},`。
- **L1301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"derfc", {{"x", DoublePrecision}}, DoublePrecision}, "erfc"},`.
  **L1301 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"derfc", {{"x", DoublePrecision}}, DoublePrecision}, "erfc"},`。
- **L1302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"derfc_scaled", {{"x", DoublePrecision}}, DoublePrecision},`.
  **L1302 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"derfc_scaled", {{"x", DoublePrecision}}, DoublePrecision},`。
- **L1303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"erfc_scaled"},`.
  **L1303 CN**: 继续一个多行参数列表、初始化器或聚合项：`"erfc_scaled"},`。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dexp", {{"x", DoublePrecision}}, DoublePrecision}, "exp"},`.
  **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dexp", {{"x", DoublePrecision}}, DoublePrecision}, "exp"},`。
- **L1305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dfloat", {{"a", AnyInt}}, DoublePrecision}, "real", true},`.
  **L1305 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dfloat", {{"a", AnyInt}}, DoublePrecision}, "real", true},`。
- **L1306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dim", {{"x", DefaultReal}, {"y", DefaultReal}}, DefaultReal}},`.
  **L1306 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dim", {{"x", DefaultReal}, {"y", DefaultReal}}, DefaultReal}},`。
- **L1307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dimag", {{"z", DoublePrecisionComplex}}, DoublePrecision}, "aimag"},`.
  **L1307 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dimag", {{"z", DoublePrecisionComplex}}, DoublePrecision}, "aimag"},`。
- **L1308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dint", {{"a", DoublePrecision}}, DoublePrecision}, "aint"},`.
  **L1308 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dint", {{"a", DoublePrecision}}, DoublePrecision}, "aint"},`。
- **L1309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dlog", {{"x", DoublePrecision}}, DoublePrecision}, "log"},`.
  **L1309 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dlog", {{"x", DoublePrecision}}, DoublePrecision}, "log"},`。
- **L1310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dlog10", {{"x", DoublePrecision}}, DoublePrecision}, "log10"},`.
  **L1310 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dlog10", {{"x", DoublePrecision}}, DoublePrecision}, "log10"},`。
- **L1311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dmax1",`.
  **L1311 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dmax1",`。
- **L1312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DoublePrecision}, {"a2", DoublePrecision},`.
  **L1312 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DoublePrecision}, {"a2", DoublePrecision},`。
- **L1313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DoublePrecision, Rank::elemental, Optionality::repeats}},`.
  **L1313 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DoublePrecision, Rank::elemental, Optionality::repeats}},`。
- **L1314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DoublePrecision},`.
  **L1314 CN**: 继续一个多行参数列表、初始化器或聚合项：`DoublePrecision},`。
- **L1315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"max", true, true},`.
  **L1315 CN**: 继续一个多行参数列表、初始化器或聚合项：`"max", true, true},`。
- **L1316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dmin1",`.
  **L1316 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dmin1",`。
- **L1317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DoublePrecision}, {"a2", DoublePrecision},`.
  **L1317 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DoublePrecision}, {"a2", DoublePrecision},`。
- **L1318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DoublePrecision, Rank::elemental, Optionality::repeats}},`.
  **L1318 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DoublePrecision, Rank::elemental, Optionality::repeats}},`。
- **L1319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DoublePrecision},`.
  **L1319 CN**: 继续一个多行参数列表、初始化器或聚合项：`DoublePrecision},`。
- **L1320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"min", true, true},`.
  **L1320 CN**: 继续一个多行参数列表、初始化器或聚合项：`"min", true, true},`。

### Lines 1321-1344

````cpp
    {{"dmod", {{"a", DoublePrecision}, {"p", DoublePrecision}},
         DoublePrecision},
        "mod"},
    {{"dnint", {{"a", DoublePrecision}}, DoublePrecision}, "anint"},
    {{"dprod", {{"x", DefaultReal}, {"y", DefaultReal}}, DoublePrecision}},
    {{"dreal", {{"a", AnyComplex}}, DoublePrecision}, "real", true},
    {{"dsign", {{"a", DoublePrecision}, {"b", DoublePrecision}},
         DoublePrecision},
        "sign"},
    {{"dsin", {{"x", DoublePrecision}}, DoublePrecision}, "sin"},
    {{"dsinh", {{"x", DoublePrecision}}, DoublePrecision}, "sinh"},
    {{"dsqrt", {{"x", DoublePrecision}}, DoublePrecision}, "sqrt"},
    {{"dtan", {{"x", DoublePrecision}}, DoublePrecision}, "tan"},
    {{"dtanh", {{"x", DoublePrecision}}, DoublePrecision}, "tanh"},
    {{"exp", {{"x", DefaultReal}}, DefaultReal}},
    {{"float", {{"a", AnyInt}}, DefaultReal}, "real", true},
    {{"iabs", {{"a", DefaultInt}}, DefaultInt}, "abs"},
    {{"idim", {{"x", DefaultInt}, {"y", DefaultInt}}, DefaultInt}, "dim"},
    {{"idint", {{"a", AnyReal}}, DefaultInt}, "int", true},
    {{"idnint", {{"a", DoublePrecision}}, DefaultInt}, "nint"},
    {{"ifix", {{"a", AnyReal}}, DefaultInt}, "int", true},
    {{"iiabs", {{"a", TypePattern{IntType, KindCode::exactKind, 2}}},
         TypePattern{IntType, KindCode::exactKind, 2}},
        "abs"},
````
- **L1321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dmod", {{"a", DoublePrecision}, {"p", DoublePrecision}},`.
  **L1321 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dmod", {{"a", DoublePrecision}, {"p", DoublePrecision}},`。
- **L1322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DoublePrecision},`.
  **L1322 CN**: 继续一个多行参数列表、初始化器或聚合项：`DoublePrecision},`。
- **L1323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"mod"},`.
  **L1323 CN**: 继续一个多行参数列表、初始化器或聚合项：`"mod"},`。
- **L1324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dnint", {{"a", DoublePrecision}}, DoublePrecision}, "anint"},`.
  **L1324 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dnint", {{"a", DoublePrecision}}, DoublePrecision}, "anint"},`。
- **L1325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dprod", {{"x", DefaultReal}, {"y", DefaultReal}}, DoublePrecision}},`.
  **L1325 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dprod", {{"x", DefaultReal}, {"y", DefaultReal}}, DoublePrecision}},`。
- **L1326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dreal", {{"a", AnyComplex}}, DoublePrecision}, "real", true},`.
  **L1326 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dreal", {{"a", AnyComplex}}, DoublePrecision}, "real", true},`。
- **L1327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dsign", {{"a", DoublePrecision}, {"b", DoublePrecision}},`.
  **L1327 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dsign", {{"a", DoublePrecision}, {"b", DoublePrecision}},`。
- **L1328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DoublePrecision},`.
  **L1328 CN**: 继续一个多行参数列表、初始化器或聚合项：`DoublePrecision},`。
- **L1329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"sign"},`.
  **L1329 CN**: 继续一个多行参数列表、初始化器或聚合项：`"sign"},`。
- **L1330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dsin", {{"x", DoublePrecision}}, DoublePrecision}, "sin"},`.
  **L1330 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dsin", {{"x", DoublePrecision}}, DoublePrecision}, "sin"},`。
- **L1331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dsinh", {{"x", DoublePrecision}}, DoublePrecision}, "sinh"},`.
  **L1331 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dsinh", {{"x", DoublePrecision}}, DoublePrecision}, "sinh"},`。
- **L1332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dsqrt", {{"x", DoublePrecision}}, DoublePrecision}, "sqrt"},`.
  **L1332 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dsqrt", {{"x", DoublePrecision}}, DoublePrecision}, "sqrt"},`。
- **L1333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dtan", {{"x", DoublePrecision}}, DoublePrecision}, "tan"},`.
  **L1333 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dtan", {{"x", DoublePrecision}}, DoublePrecision}, "tan"},`。
- **L1334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"dtanh", {{"x", DoublePrecision}}, DoublePrecision}, "tanh"},`.
  **L1334 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"dtanh", {{"x", DoublePrecision}}, DoublePrecision}, "tanh"},`。
- **L1335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"exp", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1335 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"exp", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"float", {{"a", AnyInt}}, DefaultReal}, "real", true},`.
  **L1336 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"float", {{"a", AnyInt}}, DefaultReal}, "real", true},`。
- **L1337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"iabs", {{"a", DefaultInt}}, DefaultInt}, "abs"},`.
  **L1337 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"iabs", {{"a", DefaultInt}}, DefaultInt}, "abs"},`。
- **L1338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"idim", {{"x", DefaultInt}, {"y", DefaultInt}}, DefaultInt}, "dim"},`.
  **L1338 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"idim", {{"x", DefaultInt}, {"y", DefaultInt}}, DefaultInt}, "dim"},`。
- **L1339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"idint", {{"a", AnyReal}}, DefaultInt}, "int", true},`.
  **L1339 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"idint", {{"a", AnyReal}}, DefaultInt}, "int", true},`。
- **L1340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"idnint", {{"a", DoublePrecision}}, DefaultInt}, "nint"},`.
  **L1340 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"idnint", {{"a", DoublePrecision}}, DefaultInt}, "nint"},`。
- **L1341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"ifix", {{"a", AnyReal}}, DefaultInt}, "int", true},`.
  **L1341 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"ifix", {{"a", AnyReal}}, DefaultInt}, "int", true},`。
- **L1342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"iiabs", {{"a", TypePattern{IntType, KindCode::exactKind, 2}}},`.
  **L1342 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"iiabs", {{"a", TypePattern{IntType, KindCode::exactKind, 2}}},`。
- **L1343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 2}},`.
  **L1343 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 2}},`。
- **L1344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"abs"},`.
  **L1344 CN**: 继续一个多行参数列表、初始化器或聚合项：`"abs"},`。

### Lines 1345-1368

````cpp
    // The definition of the unrestricted specific intrinsic function INDEX
    // in F'77 and F'90 has only two arguments; later standards omit the
    // argument information for all unrestricted specific intrinsic
    // procedures.  No compiler supports an implementation that allows
    // INDEX with BACK= to work when associated as an actual procedure or
    // procedure pointer target.
    {{"index", {{"string", DefaultChar}, {"substring", DefaultChar}},
        DefaultInt}},
    {{"isign", {{"a", DefaultInt}, {"b", DefaultInt}}, DefaultInt}, "sign"},
    {{"jiabs", {{"a", TypePattern{IntType, KindCode::exactKind, 4}}},
         TypePattern{IntType, KindCode::exactKind, 4}},
        "abs"},
    {{"kiabs", {{"a", TypePattern{IntType, KindCode::exactKind, 8}}},
         TypePattern{IntType, KindCode::exactKind, 8}},
        "abs"},
    {{"kidnnt", {{"a", DoublePrecision}},
         TypePattern{IntType, KindCode::exactKind, 8}},
        "nint"},
    {{"knint", {{"a", DefaultReal}},
         TypePattern{IntType, KindCode::exactKind, 8}},
        "nint"},
    {{"len", {{"string", DefaultChar, Rank::anyOrAssumedRank}}, DefaultInt,
        Rank::scalar, IntrinsicClass::inquiryFunction}},
    {{"lge", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},
````
- **L1345 EN**: Comment explains nearby logic, intent, or metadata: `The definition of the unrestricted specific intrinsic function INDEX`.
  **L1345 CN**: 注释说明附近代码的逻辑、意图或元数据：`The definition of the unrestricted specific intrinsic function INDEX`。
- **L1346 EN**: Comment explains nearby logic, intent, or metadata: `in F'77 and F'90 has only two arguments; later standards omit the`.
  **L1346 CN**: 注释说明附近代码的逻辑、意图或元数据：`in F'77 and F'90 has only two arguments; later standards omit the`。
- **L1347 EN**: Comment explains nearby logic, intent, or metadata: `argument information for all unrestricted specific intrinsic`.
  **L1347 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument information for all unrestricted specific intrinsic`。
- **L1348 EN**: Comment explains nearby logic, intent, or metadata: `procedures.  No compiler supports an implementation that allows`.
  **L1348 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedures.  No compiler supports an implementation that allows`。
- **L1349 EN**: Comment explains nearby logic, intent, or metadata: `INDEX with BACK= to work when associated as an actual procedure or`.
  **L1349 CN**: 注释说明附近代码的逻辑、意图或元数据：`INDEX with BACK= to work when associated as an actual procedure or`。
- **L1350 EN**: Comment explains nearby logic, intent, or metadata: `procedure pointer target.`.
  **L1350 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure pointer target.`。
- **L1351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"index", {{"string", DefaultChar}, {"substring", DefaultChar}},`.
  **L1351 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"index", {{"string", DefaultChar}, {"substring", DefaultChar}},`。
- **L1352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt}},`.
  **L1352 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt}},`。
- **L1353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"isign", {{"a", DefaultInt}, {"b", DefaultInt}}, DefaultInt}, "sign"},`.
  **L1353 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"isign", {{"a", DefaultInt}, {"b", DefaultInt}}, DefaultInt}, "sign"},`。
- **L1354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"jiabs", {{"a", TypePattern{IntType, KindCode::exactKind, 4}}},`.
  **L1354 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"jiabs", {{"a", TypePattern{IntType, KindCode::exactKind, 4}}},`。
- **L1355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 4}},`.
  **L1355 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 4}},`。
- **L1356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"abs"},`.
  **L1356 CN**: 继续一个多行参数列表、初始化器或聚合项：`"abs"},`。
- **L1357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"kiabs", {{"a", TypePattern{IntType, KindCode::exactKind, 8}}},`.
  **L1357 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"kiabs", {{"a", TypePattern{IntType, KindCode::exactKind, 8}}},`。
- **L1358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 8}},`.
  **L1358 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 8}},`。
- **L1359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"abs"},`.
  **L1359 CN**: 继续一个多行参数列表、初始化器或聚合项：`"abs"},`。
- **L1360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"kidnnt", {{"a", DoublePrecision}},`.
  **L1360 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"kidnnt", {{"a", DoublePrecision}},`。
- **L1361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 8}},`.
  **L1361 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 8}},`。
- **L1362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nint"},`.
  **L1362 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nint"},`。
- **L1363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"knint", {{"a", DefaultReal}},`.
  **L1363 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"knint", {{"a", DefaultReal}},`。
- **L1364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::exactKind, 8}},`.
  **L1364 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::exactKind, 8}},`。
- **L1365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nint"},`.
  **L1365 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nint"},`。
- **L1366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"len", {{"string", DefaultChar, Rank::anyOrAssumedRank}}, DefaultInt,`.
  **L1366 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"len", {{"string", DefaultChar, Rank::anyOrAssumedRank}}, DefaultInt,`。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, IntrinsicClass::inquiryFunction}},`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, IntrinsicClass::inquiryFunction}},`。
- **L1368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"lge", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},`.
  **L1368 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"lge", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},`。

### Lines 1369-1392

````cpp
         DefaultLogical},
        "lge", true},
    {{"lgt", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},
         DefaultLogical},
        "lgt", true},
    {{"lle", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},
         DefaultLogical},
        "lle", true},
    {{"llt", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},
         DefaultLogical},
        "llt", true},
    {{"log", {{"x", DefaultReal}}, DefaultReal}},
    {{"log10", {{"x", DefaultReal}}, DefaultReal}},
    {{"max0",
         {{"a1", DefaultInt}, {"a2", DefaultInt},
             {"a3", DefaultInt, Rank::elemental, Optionality::repeats}},
         DefaultInt},
        "max", true, true},
    {{"max1",
         {{"a1", DefaultReal}, {"a2", DefaultReal},
             {"a3", DefaultReal, Rank::elemental, Optionality::repeats}},
         DefaultInt},
        "max", true, true},
    {{"min0",
````
- **L1369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1369 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"lge", true},`.
  **L1370 CN**: 继续一个多行参数列表、初始化器或聚合项：`"lge", true},`。
- **L1371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"lgt", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},`.
  **L1371 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"lgt", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},`。
- **L1372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1372 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"lgt", true},`.
  **L1373 CN**: 继续一个多行参数列表、初始化器或聚合项：`"lgt", true},`。
- **L1374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"lle", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},`.
  **L1374 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"lle", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},`。
- **L1375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1375 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"lle", true},`.
  **L1376 CN**: 继续一个多行参数列表、初始化器或聚合项：`"lle", true},`。
- **L1377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"llt", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},`.
  **L1377 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"llt", {{"string_a", DefaultChar}, {"string_b", DefaultChar}},`。
- **L1378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLogical},`.
  **L1378 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLogical},`。
- **L1379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llt", true},`.
  **L1379 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llt", true},`。
- **L1380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"log", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1380 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"log", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"log10", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1381 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"log10", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"max0",`.
  **L1382 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"max0",`。
- **L1383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DefaultInt}, {"a2", DefaultInt},`.
  **L1383 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DefaultInt}, {"a2", DefaultInt},`。
- **L1384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DefaultInt, Rank::elemental, Optionality::repeats}},`.
  **L1384 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DefaultInt, Rank::elemental, Optionality::repeats}},`。
- **L1385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt},`.
  **L1385 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt},`。
- **L1386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"max", true, true},`.
  **L1386 CN**: 继续一个多行参数列表、初始化器或聚合项：`"max", true, true},`。
- **L1387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"max1",`.
  **L1387 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"max1",`。
- **L1388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DefaultReal}, {"a2", DefaultReal},`.
  **L1388 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DefaultReal}, {"a2", DefaultReal},`。
- **L1389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DefaultReal, Rank::elemental, Optionality::repeats}},`.
  **L1389 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DefaultReal, Rank::elemental, Optionality::repeats}},`。
- **L1390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt},`.
  **L1390 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt},`。
- **L1391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"max", true, true},`.
  **L1391 CN**: 继续一个多行参数列表、初始化器或聚合项：`"max", true, true},`。
- **L1392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"min0",`.
  **L1392 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"min0",`。

### Lines 1393-1416

````cpp
         {{"a1", DefaultInt}, {"a2", DefaultInt},
             {"a3", DefaultInt, Rank::elemental, Optionality::repeats}},
         DefaultInt},
        "min", true, true},
    {{"min1",
         {{"a1", DefaultReal}, {"a2", DefaultReal},
             {"a3", DefaultReal, Rank::elemental, Optionality::repeats}},
         DefaultInt},
        "min", true, true},
    {{"mod", {{"a", DefaultInt}, {"p", DefaultInt}}, DefaultInt}},
    {{"nint", {{"a", DefaultReal}}, DefaultInt}},
    {{"qerf", {{"x", QuadPrecision}}, QuadPrecision}, "erf"},
    {{"qerfc", {{"x", QuadPrecision}}, QuadPrecision}, "erfc"},
    {{"qerfc_scaled", {{"x", QuadPrecision}}, QuadPrecision}, "erfc_scaled"},
    {{"sign", {{"a", DefaultReal}, {"b", DefaultReal}}, DefaultReal}},
    {{"sin", {{"x", DefaultReal}}, DefaultReal}},
    {{"sinh", {{"x", DefaultReal}}, DefaultReal}},
    {{"sngl", {{"a", AnyReal}}, DefaultReal}, "real", true},
    {{"sqrt", {{"x", DefaultReal}}, DefaultReal}},
    {{"tan", {{"x", DefaultReal}}, DefaultReal}},
    {{"tanh", {{"x", DefaultReal}}, DefaultReal}},
    {{"zabs", {{"a", TypePattern{ComplexType, KindCode::exactKind, 8}}},
         TypePattern{RealType, KindCode::exactKind, 8}},
        "abs"},
````
- **L1393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DefaultInt}, {"a2", DefaultInt},`.
  **L1393 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DefaultInt}, {"a2", DefaultInt},`。
- **L1394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DefaultInt, Rank::elemental, Optionality::repeats}},`.
  **L1394 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DefaultInt, Rank::elemental, Optionality::repeats}},`。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt},`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt},`。
- **L1396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"min", true, true},`.
  **L1396 CN**: 继续一个多行参数列表、初始化器或聚合项：`"min", true, true},`。
- **L1397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"min1",`.
  **L1397 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"min1",`。
- **L1398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a1", DefaultReal}, {"a2", DefaultReal},`.
  **L1398 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a1", DefaultReal}, {"a2", DefaultReal},`。
- **L1399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a3", DefaultReal, Rank::elemental, Optionality::repeats}},`.
  **L1399 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a3", DefaultReal, Rank::elemental, Optionality::repeats}},`。
- **L1400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultInt},`.
  **L1400 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultInt},`。
- **L1401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"min", true, true},`.
  **L1401 CN**: 继续一个多行参数列表、初始化器或聚合项：`"min", true, true},`。
- **L1402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"mod", {{"a", DefaultInt}, {"p", DefaultInt}}, DefaultInt}},`.
  **L1402 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"mod", {{"a", DefaultInt}, {"p", DefaultInt}}, DefaultInt}},`。
- **L1403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"nint", {{"a", DefaultReal}}, DefaultInt}},`.
  **L1403 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"nint", {{"a", DefaultReal}}, DefaultInt}},`。
- **L1404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"qerf", {{"x", QuadPrecision}}, QuadPrecision}, "erf"},`.
  **L1404 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"qerf", {{"x", QuadPrecision}}, QuadPrecision}, "erf"},`。
- **L1405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"qerfc", {{"x", QuadPrecision}}, QuadPrecision}, "erfc"},`.
  **L1405 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"qerfc", {{"x", QuadPrecision}}, QuadPrecision}, "erfc"},`。
- **L1406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"qerfc_scaled", {{"x", QuadPrecision}}, QuadPrecision}, "erfc_scaled"},`.
  **L1406 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"qerfc_scaled", {{"x", QuadPrecision}}, QuadPrecision}, "erfc_scaled"},`。
- **L1407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"sign", {{"a", DefaultReal}, {"b", DefaultReal}}, DefaultReal}},`.
  **L1407 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"sign", {{"a", DefaultReal}, {"b", DefaultReal}}, DefaultReal}},`。
- **L1408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"sin", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1408 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"sin", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"sinh", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1409 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"sinh", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"sngl", {{"a", AnyReal}}, DefaultReal}, "real", true},`.
  **L1410 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"sngl", {{"a", AnyReal}}, DefaultReal}, "real", true},`。
- **L1411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"sqrt", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1411 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"sqrt", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"tan", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1412 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"tan", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"tanh", {{"x", DefaultReal}}, DefaultReal}},`.
  **L1413 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"tanh", {{"x", DefaultReal}}, DefaultReal}},`。
- **L1414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"zabs", {{"a", TypePattern{ComplexType, KindCode::exactKind, 8}}},`.
  **L1414 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"zabs", {{"a", TypePattern{ComplexType, KindCode::exactKind, 8}}},`。
- **L1415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{RealType, KindCode::exactKind, 8}},`.
  **L1415 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{RealType, KindCode::exactKind, 8}},`。
- **L1416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"abs"},`.
  **L1416 CN**: 继续一个多行参数列表、初始化器或聚合项：`"abs"},`。

### Lines 1417-1440

````cpp
};

// Must be sorted by name. The rank of the return value is ignored since
// subroutines are do not have a return value.
static const IntrinsicInterface intrinsicSubroutine[]{
    {"abort", {}, {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"atomic_add",
        {{"atom", AtomicInt, Rank::atom, Optionality::required,
             common::Intent::InOut},
            {"value", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"atomic_and",
        {{"atom", AtomicInt, Rank::atom, Optionality::required,
             common::Intent::InOut},
            {"value", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"atomic_cas",
        {{"atom", SameAtom, Rank::atom, Optionality::required,
````
- **L1417 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1417 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1418 EN**: Blank line separating nearby declarations or logic blocks.
  **L1418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1419 EN**: Comment explains nearby logic, intent, or metadata: `Must be sorted by name. The rank of the return value is ignored since`.
  **L1419 CN**: 注释说明附近代码的逻辑、意图或元数据：`Must be sorted by name. The rank of the return value is ignored since`。
- **L1420 EN**: Comment explains nearby logic, intent, or metadata: `subroutines are do not have a return value.`.
  **L1420 CN**: 注释说明附近代码的逻辑、意图或元数据：`subroutines are do not have a return value.`。
- **L1421 EN**: Continues the surrounding expression or declaration: `static const IntrinsicInterface intrinsicSubroutine[]{`.
  **L1421 CN**: 继续构造周围的表达式或声明：`static const IntrinsicInterface intrinsicSubroutine[]{`。
- **L1422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"abort", {}, {}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1422 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"abort", {}, {}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_add",`.
  **L1423 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_add",`。
- **L1424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", AtomicInt, Rank::atom, Optionality::required,`.
  **L1424 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", AtomicInt, Rank::atom, Optionality::required,`。
- **L1425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1425 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyInt, Rank::scalar, Optionality::required,`.
  **L1426 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyInt, Rank::scalar, Optionality::required,`。
- **L1427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1427 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1428 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1429 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1430 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_and",`.
  **L1431 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_and",`。
- **L1432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", AtomicInt, Rank::atom, Optionality::required,`.
  **L1432 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", AtomicInt, Rank::atom, Optionality::required,`。
- **L1433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1433 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyInt, Rank::scalar, Optionality::required,`.
  **L1434 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyInt, Rank::scalar, Optionality::required,`。
- **L1435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1435 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1436 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1437 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1438 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_cas",`.
  **L1439 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_cas",`。
- **L1440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", SameAtom, Rank::atom, Optionality::required,`.
  **L1440 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", SameAtom, Rank::atom, Optionality::required,`。

### Lines 1441-1464

````cpp
             common::Intent::InOut},
            {"old", SameAtom, Rank::scalar, Optionality::required,
                common::Intent::Out},
            {"compare", SameAtom, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"new", SameAtom, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"atomic_define",
        {{"atom", AtomicIntOrLogical, Rank::atom, Optionality::required,
             common::Intent::Out},
            {"value", AnyIntOrLogical, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"atomic_fetch_add",
        {{"atom", AtomicInt, Rank::atom, Optionality::required,
             common::Intent::InOut},
            {"value", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"old", AtomicInt, Rank::scalar, Optionality::required,
````
- **L1441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1441 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"old", SameAtom, Rank::scalar, Optionality::required,`.
  **L1442 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"old", SameAtom, Rank::scalar, Optionality::required,`。
- **L1443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1443 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"compare", SameAtom, Rank::scalar, Optionality::required,`.
  **L1444 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"compare", SameAtom, Rank::scalar, Optionality::required,`。
- **L1445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1445 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"new", SameAtom, Rank::scalar, Optionality::required,`.
  **L1446 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"new", SameAtom, Rank::scalar, Optionality::required,`。
- **L1447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1447 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1448 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1449 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1450 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_define",`.
  **L1451 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_define",`。
- **L1452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", AtomicIntOrLogical, Rank::atom, Optionality::required,`.
  **L1452 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", AtomicIntOrLogical, Rank::atom, Optionality::required,`。
- **L1453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1453 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyIntOrLogical, Rank::scalar, Optionality::required,`.
  **L1454 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyIntOrLogical, Rank::scalar, Optionality::required,`。
- **L1455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1455 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1456 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1457 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1458 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_fetch_add",`.
  **L1459 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_fetch_add",`。
- **L1460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", AtomicInt, Rank::atom, Optionality::required,`.
  **L1460 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", AtomicInt, Rank::atom, Optionality::required,`。
- **L1461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1461 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyInt, Rank::scalar, Optionality::required,`.
  **L1462 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyInt, Rank::scalar, Optionality::required,`。
- **L1463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1463 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"old", AtomicInt, Rank::scalar, Optionality::required,`.
  **L1464 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"old", AtomicInt, Rank::scalar, Optionality::required,`。

### Lines 1465-1488

````cpp
                common::Intent::Out},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"atomic_fetch_and",
        {{"atom", AtomicInt, Rank::atom, Optionality::required,
             common::Intent::InOut},
            {"value", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"old", AtomicInt, Rank::scalar, Optionality::required,
                common::Intent::Out},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"atomic_fetch_or",
        {{"atom", AtomicInt, Rank::atom, Optionality::required,
             common::Intent::InOut},
            {"value", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"old", AtomicInt, Rank::scalar, Optionality::required,
                common::Intent::Out},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
````
- **L1465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1465 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1466 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1467 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1468 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_fetch_and",`.
  **L1469 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_fetch_and",`。
- **L1470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", AtomicInt, Rank::atom, Optionality::required,`.
  **L1470 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", AtomicInt, Rank::atom, Optionality::required,`。
- **L1471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1471 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyInt, Rank::scalar, Optionality::required,`.
  **L1472 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyInt, Rank::scalar, Optionality::required,`。
- **L1473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1473 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"old", AtomicInt, Rank::scalar, Optionality::required,`.
  **L1474 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"old", AtomicInt, Rank::scalar, Optionality::required,`。
- **L1475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1475 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1476 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1477 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1478 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_fetch_or",`.
  **L1479 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_fetch_or",`。
- **L1480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", AtomicInt, Rank::atom, Optionality::required,`.
  **L1480 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", AtomicInt, Rank::atom, Optionality::required,`。
- **L1481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1481 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyInt, Rank::scalar, Optionality::required,`.
  **L1482 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyInt, Rank::scalar, Optionality::required,`。
- **L1483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1483 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"old", AtomicInt, Rank::scalar, Optionality::required,`.
  **L1484 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"old", AtomicInt, Rank::scalar, Optionality::required,`。
- **L1485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1485 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1486 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1487 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1488 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。

### Lines 1489-1512

````cpp
    {"atomic_fetch_xor",
        {{"atom", AtomicInt, Rank::atom, Optionality::required,
             common::Intent::InOut},
            {"value", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"old", AtomicInt, Rank::scalar, Optionality::required,
                common::Intent::Out},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"atomic_or",
        {{"atom", AtomicInt, Rank::atom, Optionality::required,
             common::Intent::InOut},
            {"value", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"atomic_ref",
        {{"value", AnyIntOrLogical, Rank::scalar, Optionality::required,
             common::Intent::Out},
            {"atom", AtomicIntOrLogical, Rank::atom, Optionality::required,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
````
- **L1489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_fetch_xor",`.
  **L1489 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_fetch_xor",`。
- **L1490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", AtomicInt, Rank::atom, Optionality::required,`.
  **L1490 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", AtomicInt, Rank::atom, Optionality::required,`。
- **L1491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1491 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyInt, Rank::scalar, Optionality::required,`.
  **L1492 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyInt, Rank::scalar, Optionality::required,`。
- **L1493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1493 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"old", AtomicInt, Rank::scalar, Optionality::required,`.
  **L1494 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"old", AtomicInt, Rank::scalar, Optionality::required,`。
- **L1495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1495 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1496 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1497 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1498 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_or",`.
  **L1499 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_or",`。
- **L1500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", AtomicInt, Rank::atom, Optionality::required,`.
  **L1500 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", AtomicInt, Rank::atom, Optionality::required,`。
- **L1501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1501 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyInt, Rank::scalar, Optionality::required,`.
  **L1502 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyInt, Rank::scalar, Optionality::required,`。
- **L1503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1503 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1504 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1505 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1506 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_ref",`.
  **L1507 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_ref",`。
- **L1508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"value", AnyIntOrLogical, Rank::scalar, Optionality::required,`.
  **L1508 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"value", AnyIntOrLogical, Rank::scalar, Optionality::required,`。
- **L1509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1509 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atom", AtomicIntOrLogical, Rank::atom, Optionality::required,`.
  **L1510 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atom", AtomicIntOrLogical, Rank::atom, Optionality::required,`。
- **L1511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1511 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1512 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。

### Lines 1513-1536

````cpp
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"atomic_xor",
        {{"atom", AtomicInt, Rank::atom, Optionality::required,
             common::Intent::InOut},
            {"value", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::atomicSubroutine},
    {"chdir",
        {{"name", DefaultChar, Rank::scalar, Optionality::required},
            {"status", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"co_broadcast",
        {{"a", AnyData, Rank::anyOrAssumedRank, Optionality::required,
             common::Intent::InOut},
            {"source_image", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"errmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
````
- **L1513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1513 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1514 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"atomic_xor",`.
  **L1515 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"atomic_xor",`。
- **L1516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"atom", AtomicInt, Rank::atom, Optionality::required,`.
  **L1516 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"atom", AtomicInt, Rank::atom, Optionality::required,`。
- **L1517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1517 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", AnyInt, Rank::scalar, Optionality::required,`.
  **L1518 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", AnyInt, Rank::scalar, Optionality::required,`。
- **L1519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1519 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1520 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1521 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`.
  **L1522 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::atomicSubroutine},`。
- **L1523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"chdir",`.
  **L1523 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"chdir",`。
- **L1524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"name", DefaultChar, Rank::scalar, Optionality::required},`.
  **L1524 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"name", DefaultChar, Rank::scalar, Optionality::required},`。
- **L1525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1525 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1526 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1527 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"co_broadcast",`.
  **L1528 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"co_broadcast",`。
- **L1529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", AnyData, Rank::anyOrAssumedRank, Optionality::required,`.
  **L1529 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", AnyData, Rank::anyOrAssumedRank, Optionality::required,`。
- **L1530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1530 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"source_image", AnyInt, Rank::scalar, Optionality::required,`.
  **L1531 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"source_image", AnyInt, Rank::scalar, Optionality::required,`。
- **L1532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1532 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1533 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1534 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1535 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1536 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。

### Lines 1537-1560

````cpp
        {}, Rank::elemental, IntrinsicClass::collectiveSubroutine},
    {"co_max",
        {{"a", AnyIntOrRealOrChar, Rank::anyOrAssumedRank,
             Optionality::required, common::Intent::InOut},
            {"result_image", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"errmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::collectiveSubroutine},
    {"co_min",
        {{"a", AnyIntOrRealOrChar, Rank::anyOrAssumedRank,
             Optionality::required, common::Intent::InOut},
            {"result_image", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"errmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::collectiveSubroutine},
    {"co_reduce",
        {{"a", AnyData, Rank::known, Optionality::required,
             common::Intent::InOut},
````
- **L1537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`.
  **L1537 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`。
- **L1538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"co_max",`.
  **L1538 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"co_max",`。
- **L1539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", AnyIntOrRealOrChar, Rank::anyOrAssumedRank,`.
  **L1539 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", AnyIntOrRealOrChar, Rank::anyOrAssumedRank,`。
- **L1540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::required, common::Intent::InOut},`.
  **L1540 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::required, common::Intent::InOut},`。
- **L1541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"result_image", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1541 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"result_image", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1542 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1543 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1544 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1545 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1546 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`.
  **L1547 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`。
- **L1548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"co_min",`.
  **L1548 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"co_min",`。
- **L1549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", AnyIntOrRealOrChar, Rank::anyOrAssumedRank,`.
  **L1549 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", AnyIntOrRealOrChar, Rank::anyOrAssumedRank,`。
- **L1550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::required, common::Intent::InOut},`.
  **L1550 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::required, common::Intent::InOut},`。
- **L1551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"result_image", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1551 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"result_image", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1552 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1553 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1554 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1555 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1556 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`.
  **L1557 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`。
- **L1558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"co_reduce",`.
  **L1558 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"co_reduce",`。
- **L1559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", AnyData, Rank::known, Optionality::required,`.
  **L1559 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", AnyData, Rank::known, Optionality::required,`。
- **L1560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1560 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。

### Lines 1561-1584

````cpp
            {"operation", SameType, Rank::reduceOperation},
            {"result_image", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"errmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::collectiveSubroutine},
    {"co_sum",
        {{"a", AnyNumeric, Rank::anyOrAssumedRank, Optionality::required,
             common::Intent::InOut},
            {"result_image", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::In},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"errmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::collectiveSubroutine},
    {"cpu_time",
        {{"time", AnyReal, Rank::scalar, Optionality::required,
            common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"date_and_time",
        {{"date", DefaultChar, Rank::scalar, Optionality::optional,
````
- **L1561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"operation", SameType, Rank::reduceOperation},`.
  **L1561 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"operation", SameType, Rank::reduceOperation},`。
- **L1562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"result_image", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1562 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"result_image", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1563 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1564 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1565 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1566 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1567 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`.
  **L1568 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`。
- **L1569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"co_sum",`.
  **L1569 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"co_sum",`。
- **L1570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"a", AnyNumeric, Rank::anyOrAssumedRank, Optionality::required,`.
  **L1570 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"a", AnyNumeric, Rank::anyOrAssumedRank, Optionality::required,`。
- **L1571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1571 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"result_image", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1572 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"result_image", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1573 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1574 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1576 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1577 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`.
  **L1578 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::collectiveSubroutine},`。
- **L1579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cpu_time",`.
  **L1579 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cpu_time",`。
- **L1580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"time", AnyReal, Rank::scalar, Optionality::required,`.
  **L1580 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"time", AnyReal, Rank::scalar, Optionality::required,`。
- **L1581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1581 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1582 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"date_and_time",`.
  **L1583 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"date_and_time",`。
- **L1584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"date", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1584 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"date", DefaultChar, Rank::scalar, Optionality::optional,`。

### Lines 1585-1608

````cpp
             common::Intent::Out},
            {"time", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"zone", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"values", AnyInt, Rank::vector, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"etime",
        {{"values", TypePattern{RealType, KindCode::exactKind, 4}, Rank::vector,
             Optionality::required, common::Intent::Out},
            {"time", TypePattern{RealType, KindCode::exactKind, 4},
                Rank::scalar, Optionality::required, common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"event_query",
        {{"event", EventType, Rank::scalar},
            {"count", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::Out},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"execute_command_line",
        {{"command", DefaultChar, Rank::scalar},
            {"wait", AnyLogical, Rank::scalar, Optionality::optional},
````
- **L1585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1585 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"time", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1586 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"time", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1587 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"zone", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1588 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"zone", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1589 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"values", AnyInt, Rank::vector, Optionality::optional,`.
  **L1590 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"values", AnyInt, Rank::vector, Optionality::optional,`。
- **L1591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1591 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1592 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"etime",`.
  **L1593 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"etime",`。
- **L1594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"values", TypePattern{RealType, KindCode::exactKind, 4}, Rank::vector,`.
  **L1594 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"values", TypePattern{RealType, KindCode::exactKind, 4}, Rank::vector,`。
- **L1595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::required, common::Intent::Out},`.
  **L1595 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::required, common::Intent::Out},`。
- **L1596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"time", TypePattern{RealType, KindCode::exactKind, 4},`.
  **L1596 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"time", TypePattern{RealType, KindCode::exactKind, 4},`。
- **L1597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, Optionality::required, common::Intent::Out}},`.
  **L1597 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, Optionality::required, common::Intent::Out}},`。
- **L1598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1598 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"event_query",`.
  **L1599 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"event_query",`。
- **L1600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"event", EventType, Rank::scalar},`.
  **L1600 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"event", EventType, Rank::scalar},`。
- **L1601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"count", AnyInt, Rank::scalar, Optionality::required,`.
  **L1601 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"count", AnyInt, Rank::scalar, Optionality::required,`。
- **L1602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1602 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1603 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1604 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1605 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"execute_command_line",`.
  **L1606 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"execute_command_line",`。
- **L1607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"command", DefaultChar, Rank::scalar},`.
  **L1607 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"command", DefaultChar, Rank::scalar},`。
- **L1608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"wait", AnyLogical, Rank::scalar, Optionality::optional},`.
  **L1608 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"wait", AnyLogical, Rank::scalar, Optionality::optional},`。

### Lines 1609-1632

````cpp
            {"exitstat",
                TypePattern{IntType, KindCode::greaterOrEqualToKind, 4},
                Rank::scalar, Optionality::optional, common::Intent::InOut},
            {"cmdstat", TypePattern{IntType, KindCode::greaterOrEqualToKind, 2},
                Rank::scalar, Optionality::optional, common::Intent::Out},
            {"cmdmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"exit", {{"status", AnyInt, Rank::scalar, Optionality::optional}}, {},
        Rank::elemental, IntrinsicClass::impureSubroutine},
    {"free", {{"ptr", Addressable}}, {}},
    {"flush",
        {{"unit", AnyInt, Rank::scalar, Optionality::optional,
            common::Intent::In}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"fseek",
        {{"unit", AnyInt, Rank::scalar}, {"offset", AnyInt, Rank::scalar},
            {"whence", AnyInt, Rank::scalar},
            {"status", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"ftell",
        {{"unit", AnyInt, Rank::scalar},
            {"offset", AnyInt, Rank::scalar, Optionality::required,
````
- **L1609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"exitstat",`.
  **L1609 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"exitstat",`。
- **L1610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypePattern{IntType, KindCode::greaterOrEqualToKind, 4},`.
  **L1610 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypePattern{IntType, KindCode::greaterOrEqualToKind, 4},`。
- **L1611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, Optionality::optional, common::Intent::InOut},`.
  **L1611 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, Optionality::optional, common::Intent::InOut},`。
- **L1612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cmdstat", TypePattern{IntType, KindCode::greaterOrEqualToKind, 2},`.
  **L1612 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cmdstat", TypePattern{IntType, KindCode::greaterOrEqualToKind, 2},`。
- **L1613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, Optionality::optional, common::Intent::Out},`.
  **L1613 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, Optionality::optional, common::Intent::Out},`。
- **L1614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"cmdmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1614 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"cmdmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1615 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1616 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"exit", {{"status", AnyInt, Rank::scalar, Optionality::optional}}, {},`.
  **L1617 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"exit", {{"status", AnyInt, Rank::scalar, Optionality::optional}}, {},`。
- **L1618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1618 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"free", {{"ptr", Addressable}}, {}},`.
  **L1619 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"free", {{"ptr", Addressable}}, {}},`。
- **L1620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"flush",`.
  **L1620 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"flush",`。
- **L1621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"unit", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1621 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"unit", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In}},`.
  **L1622 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In}},`。
- **L1623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1623 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"fseek",`.
  **L1624 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"fseek",`。
- **L1625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"unit", AnyInt, Rank::scalar}, {"offset", AnyInt, Rank::scalar},`.
  **L1625 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"unit", AnyInt, Rank::scalar}, {"offset", AnyInt, Rank::scalar},`。
- **L1626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"whence", AnyInt, Rank::scalar},`.
  **L1626 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"whence", AnyInt, Rank::scalar},`。
- **L1627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1627 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1628 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ftell",`.
  **L1630 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ftell",`。
- **L1631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"unit", AnyInt, Rank::scalar},`.
  **L1631 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"unit", AnyInt, Rank::scalar},`。
- **L1632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"offset", AnyInt, Rank::scalar, Optionality::required,`.
  **L1632 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"offset", AnyInt, Rank::scalar, Optionality::required,`。

### Lines 1633-1656

````cpp
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"get_command",
        {{"command", DefaultChar, Rank::scalar, Optionality::optional,
             common::Intent::Out},
            {"length", TypePattern{IntType, KindCode::greaterOrEqualToKind, 2},
                Rank::scalar, Optionality::optional, common::Intent::Out},
            {"status", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"errmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"get_command_argument",
        {{"number", AnyInt, Rank::scalar},
            {"value", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"length", TypePattern{IntType, KindCode::greaterOrEqualToKind, 2},
                Rank::scalar, Optionality::optional, common::Intent::Out},
            {"status", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"errmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"get_environment_variable",
````
- **L1633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1633 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1634 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"get_command",`.
  **L1635 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"get_command",`。
- **L1636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"command", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1636 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"command", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1637 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"length", TypePattern{IntType, KindCode::greaterOrEqualToKind, 2},`.
  **L1638 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"length", TypePattern{IntType, KindCode::greaterOrEqualToKind, 2},`。
- **L1639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, Optionality::optional, common::Intent::Out},`.
  **L1639 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, Optionality::optional, common::Intent::Out},`。
- **L1640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1640 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1641 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1642 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1643 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1644 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"get_command_argument",`.
  **L1645 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"get_command_argument",`。
- **L1646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"number", AnyInt, Rank::scalar},`.
  **L1646 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"number", AnyInt, Rank::scalar},`。
- **L1647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1647 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1648 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"length", TypePattern{IntType, KindCode::greaterOrEqualToKind, 2},`.
  **L1649 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"length", TypePattern{IntType, KindCode::greaterOrEqualToKind, 2},`。
- **L1650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, Optionality::optional, common::Intent::Out},`.
  **L1650 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, Optionality::optional, common::Intent::Out},`。
- **L1651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1651 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1652 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1653 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1654 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1655 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"get_environment_variable",`.
  **L1656 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"get_environment_variable",`。

### Lines 1657-1680

````cpp
        {{"name", DefaultChar, Rank::scalar},
            {"value", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"length", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"status", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"trim_name", AnyLogical, Rank::scalar, Optionality::optional},
            {"errmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"getcwd",
        {{"c", DefaultChar, Rank::scalar, Optionality::required,
             common::Intent::Out},
            {"status", TypePattern{IntType, KindCode::greaterOrEqualToKind, 4},
                Rank::scalar, Optionality::optional, common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"hostnm",
        {{"c", DefaultChar, Rank::scalar, Optionality::required,
             common::Intent::Out},
            {"status", TypePattern{IntType, KindCode::greaterOrEqualToKind, 4},
                Rank::scalar, Optionality::optional, common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"move_alloc",
````
- **L1657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"name", DefaultChar, Rank::scalar},`.
  **L1657 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"name", DefaultChar, Rank::scalar},`。
- **L1658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"value", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1658 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"value", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1659 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"length", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1660 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"length", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1661 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1662 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1663 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"trim_name", AnyLogical, Rank::scalar, Optionality::optional},`.
  **L1664 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"trim_name", AnyLogical, Rank::scalar, Optionality::optional},`。
- **L1665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1665 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1666 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1667 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"getcwd",`.
  **L1668 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"getcwd",`。
- **L1669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"c", DefaultChar, Rank::scalar, Optionality::required,`.
  **L1669 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"c", DefaultChar, Rank::scalar, Optionality::required,`。
- **L1670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1670 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", TypePattern{IntType, KindCode::greaterOrEqualToKind, 4},`.
  **L1671 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", TypePattern{IntType, KindCode::greaterOrEqualToKind, 4},`。
- **L1672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, Optionality::optional, common::Intent::Out}},`.
  **L1672 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, Optionality::optional, common::Intent::Out}},`。
- **L1673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1673 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"hostnm",`.
  **L1674 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"hostnm",`。
- **L1675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"c", DefaultChar, Rank::scalar, Optionality::required,`.
  **L1675 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"c", DefaultChar, Rank::scalar, Optionality::required,`。
- **L1676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1676 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", TypePattern{IntType, KindCode::greaterOrEqualToKind, 4},`.
  **L1677 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", TypePattern{IntType, KindCode::greaterOrEqualToKind, 4},`。
- **L1678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::scalar, Optionality::optional, common::Intent::Out}},`.
  **L1678 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::scalar, Optionality::optional, common::Intent::Out}},`。
- **L1679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1679 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"move_alloc",`.
  **L1680 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"move_alloc",`。

### Lines 1681-1704

````cpp
        {{"from", SameType, Rank::known, Optionality::required,
             common::Intent::InOut},
            {"to", SameType, Rank::known, Optionality::required,
                common::Intent::Out},
            {"stat", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"errmsg", DefaultChar, Rank::scalar, Optionality::optional,
                common::Intent::InOut}},
        {}, Rank::elemental, IntrinsicClass::pureSubroutine},
    {"perror", {{"string", DefaultChar, Rank::scalar}}, {}, Rank::elemental,
        IntrinsicClass::impureSubroutine},
    {"putenv",
        {{"str", DefaultChar, Rank::scalar, Optionality::required,
             common::Intent::In},
            {"status", DefaultInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"mvbits",
        {{"from", SameIntOrUnsigned}, {"frompos", AnyInt}, {"len", AnyInt},
            {"to", SameIntOrUnsigned, Rank::elemental, Optionality::required,
                common::Intent::Out},
            {"topos", AnyInt}},
        {}, Rank::elemental, IntrinsicClass::elementalSubroutine},
    {"random_init",
````
- **L1681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"from", SameType, Rank::known, Optionality::required,`.
  **L1681 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"from", SameType, Rank::known, Optionality::required,`。
- **L1682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1682 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"to", SameType, Rank::known, Optionality::required,`.
  **L1683 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"to", SameType, Rank::known, Optionality::required,`。
- **L1684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1684 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"stat", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1685 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"stat", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1686 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`.
  **L1687 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"errmsg", DefaultChar, Rank::scalar, Optionality::optional,`。
- **L1688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut}},`.
  **L1688 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut}},`。
- **L1689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::pureSubroutine},`.
  **L1689 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::pureSubroutine},`。
- **L1690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"perror", {{"string", DefaultChar, Rank::scalar}}, {}, Rank::elemental,`.
  **L1690 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"perror", {{"string", DefaultChar, Rank::scalar}}, {}, Rank::elemental,`。
- **L1691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::impureSubroutine},`.
  **L1691 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::impureSubroutine},`。
- **L1692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"putenv",`.
  **L1692 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"putenv",`。
- **L1693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"str", DefaultChar, Rank::scalar, Optionality::required,`.
  **L1693 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"str", DefaultChar, Rank::scalar, Optionality::required,`。
- **L1694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1694 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", DefaultInt, Rank::scalar, Optionality::optional,`.
  **L1695 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", DefaultInt, Rank::scalar, Optionality::optional,`。
- **L1696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1696 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1697 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"mvbits",`.
  **L1698 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"mvbits",`。
- **L1699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"from", SameIntOrUnsigned}, {"frompos", AnyInt}, {"len", AnyInt},`.
  **L1699 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"from", SameIntOrUnsigned}, {"frompos", AnyInt}, {"len", AnyInt},`。
- **L1700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"to", SameIntOrUnsigned, Rank::elemental, Optionality::required,`.
  **L1700 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"to", SameIntOrUnsigned, Rank::elemental, Optionality::required,`。
- **L1701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1701 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"topos", AnyInt}},`.
  **L1702 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"topos", AnyInt}},`。
- **L1703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::elementalSubroutine},`.
  **L1703 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::elementalSubroutine},`。
- **L1704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"random_init",`.
  **L1704 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"random_init",`。

### Lines 1705-1728

````cpp
        {{"repeatable", AnyLogical, Rank::scalar},
            {"image_distinct", AnyLogical, Rank::scalar}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"random_number",
        {{"harvest", {RealType | UnsignedType, KindCode::any}, Rank::known,
            Optionality::required, common::Intent::Out,
            {ArgFlag::notAssumedSize}}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"random_seed",
        {{"size", DefaultInt, Rank::scalar, Optionality::optional,
             common::Intent::Out},
            {"put", DefaultInt, Rank::vector, Optionality::optional},
            {"get", DefaultInt, Rank::vector, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"rename",
        {{"path1", DefaultChar, Rank::scalar},
            {"path2", DefaultChar, Rank::scalar},
            {"status", DefaultInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::scalar, IntrinsicClass::impureSubroutine},
    {"second", {{"time", DefaultReal, Rank::scalar}}, {}, Rank::scalar,
        IntrinsicClass::impureSubroutine},
    {"__builtin_show_descriptor", {{"d", AnyData, Rank::anyOrAssumedRank}}, {},
````
- **L1705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"repeatable", AnyLogical, Rank::scalar},`.
  **L1705 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"repeatable", AnyLogical, Rank::scalar},`。
- **L1706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"image_distinct", AnyLogical, Rank::scalar}},`.
  **L1706 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"image_distinct", AnyLogical, Rank::scalar}},`。
- **L1707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1707 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"random_number",`.
  **L1708 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"random_number",`。
- **L1709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"harvest", {RealType | UnsignedType, KindCode::any}, Rank::known,`.
  **L1709 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"harvest", {RealType | UnsignedType, KindCode::any}, Rank::known,`。
- **L1710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Optionality::required, common::Intent::Out,`.
  **L1710 CN**: 继续一个多行参数列表、初始化器或聚合项：`Optionality::required, common::Intent::Out,`。
- **L1711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{ArgFlag::notAssumedSize}}},`.
  **L1711 CN**: 继续一个多行参数列表、初始化器或聚合项：`{ArgFlag::notAssumedSize}}},`。
- **L1712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1712 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"random_seed",`.
  **L1713 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"random_seed",`。
- **L1714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"size", DefaultInt, Rank::scalar, Optionality::optional,`.
  **L1714 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"size", DefaultInt, Rank::scalar, Optionality::optional,`。
- **L1715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1715 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"put", DefaultInt, Rank::vector, Optionality::optional},`.
  **L1716 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"put", DefaultInt, Rank::vector, Optionality::optional},`。
- **L1717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"get", DefaultInt, Rank::vector, Optionality::optional,`.
  **L1717 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"get", DefaultInt, Rank::vector, Optionality::optional,`。
- **L1718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1718 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1719 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"rename",`.
  **L1720 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"rename",`。
- **L1721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"path1", DefaultChar, Rank::scalar},`.
  **L1721 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"path1", DefaultChar, Rank::scalar},`。
- **L1722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"path2", DefaultChar, Rank::scalar},`.
  **L1722 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"path2", DefaultChar, Rank::scalar},`。
- **L1723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", DefaultInt, Rank::scalar, Optionality::optional,`.
  **L1723 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", DefaultInt, Rank::scalar, Optionality::optional,`。
- **L1724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1724 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::scalar, IntrinsicClass::impureSubroutine},`.
  **L1725 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::scalar, IntrinsicClass::impureSubroutine},`。
- **L1726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"second", {{"time", DefaultReal, Rank::scalar}}, {}, Rank::scalar,`.
  **L1726 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"second", {{"time", DefaultReal, Rank::scalar}}, {}, Rank::scalar,`。
- **L1727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrinsicClass::impureSubroutine},`.
  **L1727 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrinsicClass::impureSubroutine},`。
- **L1728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__builtin_show_descriptor", {{"d", AnyData, Rank::anyOrAssumedRank}}, {},`.
  **L1728 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__builtin_show_descriptor", {{"d", AnyData, Rank::anyOrAssumedRank}}, {},`。

### Lines 1729-1752

````cpp
        Rank::elemental, IntrinsicClass::impureSubroutine},
    {"system",
        {{"command", DefaultChar, Rank::scalar},
            {"exitstat", DefaultInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"system_clock",
        {{"count", AnyInt, Rank::scalar, Optionality::optional,
             common::Intent::Out},
            {"count_rate", AnyIntOrReal, Rank::scalar, Optionality::optional,
                common::Intent::Out},
            {"count_max", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"signal",
        {{"number", AnyInt, Rank::scalar, Optionality::required,
             common::Intent::In},
            // note: any pointer also accepts AnyInt
            {"handler", AnyPointer, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"status", AnyInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"sleep",
````
- **L1729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1729 CN**: 继续一个多行参数列表、初始化器或聚合项：`Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"system",`.
  **L1730 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"system",`。
- **L1731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"command", DefaultChar, Rank::scalar},`.
  **L1731 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"command", DefaultChar, Rank::scalar},`。
- **L1732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"exitstat", DefaultInt, Rank::scalar, Optionality::optional,`.
  **L1732 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"exitstat", DefaultInt, Rank::scalar, Optionality::optional,`。
- **L1733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1733 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1734 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"system_clock",`.
  **L1735 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"system_clock",`。
- **L1736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"count", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1736 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"count", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1737 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"count_rate", AnyIntOrReal, Rank::scalar, Optionality::optional,`.
  **L1738 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"count_rate", AnyIntOrReal, Rank::scalar, Optionality::optional,`。
- **L1739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1739 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"count_max", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1740 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"count_max", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1741 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1742 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"signal",`.
  **L1743 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"signal",`。
- **L1744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"number", AnyInt, Rank::scalar, Optionality::required,`.
  **L1744 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"number", AnyInt, Rank::scalar, Optionality::required,`。
- **L1745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1745 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1746 EN**: Comment explains nearby logic, intent, or metadata: `note: any pointer also accepts AnyInt`.
  **L1746 CN**: 注释说明附近代码的逻辑、意图或元数据：`note: any pointer also accepts AnyInt`。
- **L1747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"handler", AnyPointer, Rank::scalar, Optionality::required,`.
  **L1747 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"handler", AnyPointer, Rank::scalar, Optionality::required,`。
- **L1748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1748 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", AnyInt, Rank::scalar, Optionality::optional,`.
  **L1749 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", AnyInt, Rank::scalar, Optionality::optional,`。
- **L1750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1750 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1751 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"sleep",`.
  **L1752 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"sleep",`。

### Lines 1753-1776

````cpp
        {{"seconds", AnyInt, Rank::scalar, Optionality::required,
            common::Intent::In}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
    {"split",
        {{"string", SameCharNoLen, Rank::scalar, Optionality::required,
             common::Intent::In},
            {"set", SameCharNoLen, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"pos", AnyInt, Rank::scalar, Optionality::required,
                common::Intent::InOut},
            {"back", AnyLogical, Rank::scalar, Optionality::optional,
                common::Intent::In}},
        {}, Rank::elemental, IntrinsicClass::pureSubroutine},
    {"tokenize",
        {{"string", SameCharNoLen, Rank::scalar, Optionality::required,
             common::Intent::In},
            {"set", SameCharNoLen, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"tokens", SameCharNoLen, Rank::vector, Optionality::required,
                common::Intent::Out},
            {"separator", SameCharNoLen, Rank::vector, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::pureSubroutine},
    {"tokenize",
````
- **L1753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"seconds", AnyInt, Rank::scalar, Optionality::required,`.
  **L1753 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"seconds", AnyInt, Rank::scalar, Optionality::required,`。
- **L1754 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In}},`.
  **L1754 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In}},`。
- **L1755 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1755 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"split",`.
  **L1756 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"split",`。
- **L1757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"string", SameCharNoLen, Rank::scalar, Optionality::required,`.
  **L1757 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"string", SameCharNoLen, Rank::scalar, Optionality::required,`。
- **L1758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1758 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"set", SameCharNoLen, Rank::scalar, Optionality::required,`.
  **L1759 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"set", SameCharNoLen, Rank::scalar, Optionality::required,`。
- **L1760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1760 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pos", AnyInt, Rank::scalar, Optionality::required,`.
  **L1761 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pos", AnyInt, Rank::scalar, Optionality::required,`。
- **L1762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::InOut},`.
  **L1762 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::InOut},`。
- **L1763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"back", AnyLogical, Rank::scalar, Optionality::optional,`.
  **L1763 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"back", AnyLogical, Rank::scalar, Optionality::optional,`。
- **L1764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In}},`.
  **L1764 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In}},`。
- **L1765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::pureSubroutine},`.
  **L1765 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::pureSubroutine},`。
- **L1766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"tokenize",`.
  **L1766 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"tokenize",`。
- **L1767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"string", SameCharNoLen, Rank::scalar, Optionality::required,`.
  **L1767 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"string", SameCharNoLen, Rank::scalar, Optionality::required,`。
- **L1768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1768 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"set", SameCharNoLen, Rank::scalar, Optionality::required,`.
  **L1769 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"set", SameCharNoLen, Rank::scalar, Optionality::required,`。
- **L1770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1770 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"tokens", SameCharNoLen, Rank::vector, Optionality::required,`.
  **L1771 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"tokens", SameCharNoLen, Rank::vector, Optionality::required,`。
- **L1772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1772 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"separator", SameCharNoLen, Rank::vector, Optionality::optional,`.
  **L1773 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"separator", SameCharNoLen, Rank::vector, Optionality::optional,`。
- **L1774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1774 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::pureSubroutine},`.
  **L1775 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::pureSubroutine},`。
- **L1776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"tokenize",`.
  **L1776 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"tokenize",`。

### Lines 1777-1800

````cpp
        {{"string", SameCharNoLen, Rank::scalar, Optionality::required,
             common::Intent::In},
            {"set", SameCharNoLen, Rank::scalar, Optionality::required,
                common::Intent::In},
            {"first", AnyInt, Rank::vector, Optionality::required,
                common::Intent::Out},
            {"last", AnyInt, Rank::vector, Optionality::required,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::pureSubroutine},
    {"unlink",
        {{"path", DefaultChar, Rank::scalar, Optionality::required,
             common::Intent::In},
            {"status", DefaultInt, Rank::scalar, Optionality::optional,
                common::Intent::Out}},
        {}, Rank::elemental, IntrinsicClass::impureSubroutine},
};

// Finds a built-in derived type and returns it as a DynamicType.
static DynamicType GetBuiltinDerivedType(
    const semantics::Scope *builtinsScope, const char *which) {
  if (!builtinsScope) {
    common::die("INTERNAL: The __fortran_builtins module was not found, and "
                "the type '%s' was required",
        which);
````
- **L1777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"string", SameCharNoLen, Rank::scalar, Optionality::required,`.
  **L1777 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"string", SameCharNoLen, Rank::scalar, Optionality::required,`。
- **L1778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1778 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"set", SameCharNoLen, Rank::scalar, Optionality::required,`.
  **L1779 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"set", SameCharNoLen, Rank::scalar, Optionality::required,`。
- **L1780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1780 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"first", AnyInt, Rank::vector, Optionality::required,`.
  **L1781 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"first", AnyInt, Rank::vector, Optionality::required,`。
- **L1782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out},`.
  **L1782 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out},`。
- **L1783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"last", AnyInt, Rank::vector, Optionality::required,`.
  **L1783 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"last", AnyInt, Rank::vector, Optionality::required,`。
- **L1784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1784 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::pureSubroutine},`.
  **L1785 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::pureSubroutine},`。
- **L1786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"unlink",`.
  **L1786 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"unlink",`。
- **L1787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{"path", DefaultChar, Rank::scalar, Optionality::required,`.
  **L1787 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{"path", DefaultChar, Rank::scalar, Optionality::required,`。
- **L1788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::In},`.
  **L1788 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::In},`。
- **L1789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"status", DefaultInt, Rank::scalar, Optionality::optional,`.
  **L1789 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"status", DefaultInt, Rank::scalar, Optionality::optional,`。
- **L1790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Intent::Out}},`.
  **L1790 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Intent::Out}},`。
- **L1791 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{}, Rank::elemental, IntrinsicClass::impureSubroutine},`.
  **L1791 CN**: 继续一个多行参数列表、初始化器或聚合项：`{}, Rank::elemental, IntrinsicClass::impureSubroutine},`。
- **L1792 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1792 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1793 EN**: Blank line separating nearby declarations or logic blocks.
  **L1793 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1794 EN**: Comment explains nearby logic, intent, or metadata: `Finds a built-in derived type and returns it as a DynamicType.`.
  **L1794 CN**: 注释说明附近代码的逻辑、意图或元数据：`Finds a built-in derived type and returns it as a DynamicType.`。
- **L1795 EN**: Continues logic associated with callable symbol `GetBuiltinDerivedType`.
  **L1795 CN**: 继续与可调用符号 `GetBuiltinDerivedType` 相关的逻辑。
- **L1796 EN**: Continues the surrounding expression or declaration: `const semantics::Scope *builtinsScope, const char *which) {`.
  **L1796 CN**: 继续构造周围的表达式或声明：`const semantics::Scope *builtinsScope, const char *which) {`。
- **L1797 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1797 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1798 EN**: Continues logic associated with callable symbol `die`.
  **L1798 CN**: 继续与可调用符号 `die` 相关的逻辑。
- **L1799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the type '%s' was required",`.
  **L1799 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the type '%s' was required",`。
- **L1800 EN**: Executes a standalone statement or declaration: `which);`.
  **L1800 CN**: 执行一条独立语句或声明：`which);`。

### Lines 1801-1824

````cpp
  }
  auto iter{
      builtinsScope->find(semantics::SourceName{which, std::strlen(which)})};
  if (iter == builtinsScope->cend()) {
    // keep the string all together
    // clang-format off
    common::die(
        "INTERNAL: The __fortran_builtins module does not define the type '%s'",
        which);
    // clang-format on
  }
  const semantics::Symbol &symbol{*iter->second};
  const semantics::Scope &scope{DEREF(symbol.scope())};
  const semantics::DerivedTypeSpec &derived{DEREF(scope.derivedTypeSpec())};
  return DynamicType{derived};
}

static std::int64_t GetBuiltinKind(
    const semantics::Scope *builtinsScope, const char *which) {
  if (!builtinsScope) {
    common::die("INTERNAL: The __fortran_builtins module was not found, and "
                "the kind '%s' was required",
        which);
  }
````
- **L1801 EN**: Closes the current lexical scope or compound statement.
  **L1801 CN**: 结束当前词法作用域或复合语句块。
- **L1802 EN**: Continues the surrounding expression or declaration: `auto iter{`.
  **L1802 CN**: 继续构造周围的表达式或声明：`auto iter{`。
- **L1803 EN**: Executes a call or declaration centered on `builtinsScope->find`.
  **L1803 CN**: 执行以 `builtinsScope->find` 为核心的调用或声明。
- **L1804 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1804 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1805 EN**: Comment explains nearby logic, intent, or metadata: `keep the string all together`.
  **L1805 CN**: 注释说明附近代码的逻辑、意图或元数据：`keep the string all together`。
- **L1806 EN**: Comment explains nearby logic, intent, or metadata: `clang-format off`.
  **L1806 CN**: 注释说明附近代码的逻辑、意图或元数据：`clang-format off`。
- **L1807 EN**: Continues logic associated with callable symbol `die`.
  **L1807 CN**: 继续与可调用符号 `die` 相关的逻辑。
- **L1808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"INTERNAL: The __fortran_builtins module does not define the type '%s'",`.
  **L1808 CN**: 继续一个多行参数列表、初始化器或聚合项：`"INTERNAL: The __fortran_builtins module does not define the type '%s'",`。
- **L1809 EN**: Executes a standalone statement or declaration: `which);`.
  **L1809 CN**: 执行一条独立语句或声明：`which);`。
- **L1810 EN**: Comment explains nearby logic, intent, or metadata: `clang-format on`.
  **L1810 CN**: 注释说明附近代码的逻辑、意图或元数据：`clang-format on`。
- **L1811 EN**: Closes the current lexical scope or compound statement.
  **L1811 CN**: 结束当前词法作用域或复合语句块。
- **L1812 EN**: Executes a standalone statement or declaration: `const semantics::Symbol &symbol{*iter->second};`.
  **L1812 CN**: 执行一条独立语句或声明：`const semantics::Symbol &symbol{*iter->second};`。
- **L1813 EN**: Executes a call or declaration centered on `&scope{DEREF`.
  **L1813 CN**: 执行以 `&scope{DEREF` 为核心的调用或声明。
- **L1814 EN**: Executes a call or declaration centered on `&derived{DEREF`.
  **L1814 CN**: 执行以 `&derived{DEREF` 为核心的调用或声明。
- **L1815 EN**: Returns from the current function with `DynamicType{derived}`.
  **L1815 CN**: 以 `DynamicType{derived}` 从当前函数返回。
- **L1816 EN**: Closes the current lexical scope or compound statement.
  **L1816 CN**: 结束当前词法作用域或复合语句块。
- **L1817 EN**: Blank line separating nearby declarations or logic blocks.
  **L1817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1818 EN**: Continues logic associated with callable symbol `GetBuiltinKind`.
  **L1818 CN**: 继续与可调用符号 `GetBuiltinKind` 相关的逻辑。
- **L1819 EN**: Continues the surrounding expression or declaration: `const semantics::Scope *builtinsScope, const char *which) {`.
  **L1819 CN**: 继续构造周围的表达式或声明：`const semantics::Scope *builtinsScope, const char *which) {`。
- **L1820 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1820 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1821 EN**: Continues logic associated with callable symbol `die`.
  **L1821 CN**: 继续与可调用符号 `die` 相关的逻辑。
- **L1822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the kind '%s' was required",`.
  **L1822 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the kind '%s' was required",`。
- **L1823 EN**: Executes a standalone statement or declaration: `which);`.
  **L1823 CN**: 执行一条独立语句或声明：`which);`。
- **L1824 EN**: Closes the current lexical scope or compound statement.
  **L1824 CN**: 结束当前词法作用域或复合语句块。

### Lines 1825-1848

````cpp
  auto iter{
      builtinsScope->find(semantics::SourceName{which, std::strlen(which)})};
  if (iter == builtinsScope->cend()) {
    common::die(
        "INTERNAL: The __fortran_builtins module does not define the kind '%s'",
        which);
  }
  const semantics::Symbol &symbol{*iter->second};
  const auto &details{
      DEREF(symbol.detailsIf<semantics::ObjectEntityDetails>())};
  if (const auto kind{ToInt64(details.init())}) {
    return *kind;
  } else {
    common::die(
        "INTERNAL: The __fortran_builtins module does not define the kind '%s'",
        which);
    return -1;
  }
}

// Ensure that the keywords of arguments to MAX/MIN and their variants
// are of the form A123 with no duplicates or leading zeroes.
static bool CheckMaxMinArgument(parser::CharBlock keyword,
    std::set<parser::CharBlock> &set, const char *intrinsicName,
````
- **L1825 EN**: Continues the surrounding expression or declaration: `auto iter{`.
  **L1825 CN**: 继续构造周围的表达式或声明：`auto iter{`。
- **L1826 EN**: Executes a call or declaration centered on `builtinsScope->find`.
  **L1826 CN**: 执行以 `builtinsScope->find` 为核心的调用或声明。
- **L1827 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1827 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1828 EN**: Continues logic associated with callable symbol `die`.
  **L1828 CN**: 继续与可调用符号 `die` 相关的逻辑。
- **L1829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"INTERNAL: The __fortran_builtins module does not define the kind '%s'",`.
  **L1829 CN**: 继续一个多行参数列表、初始化器或聚合项：`"INTERNAL: The __fortran_builtins module does not define the kind '%s'",`。
- **L1830 EN**: Executes a standalone statement or declaration: `which);`.
  **L1830 CN**: 执行一条独立语句或声明：`which);`。
- **L1831 EN**: Closes the current lexical scope or compound statement.
  **L1831 CN**: 结束当前词法作用域或复合语句块。
- **L1832 EN**: Executes a standalone statement or declaration: `const semantics::Symbol &symbol{*iter->second};`.
  **L1832 CN**: 执行一条独立语句或声明：`const semantics::Symbol &symbol{*iter->second};`。
- **L1833 EN**: Continues the surrounding expression or declaration: `const auto &details{`.
  **L1833 CN**: 继续构造周围的表达式或声明：`const auto &details{`。
- **L1834 EN**: Executes a call or declaration centered on `DEREF`.
  **L1834 CN**: 执行以 `DEREF` 为核心的调用或声明。
- **L1835 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1835 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1836 EN**: Returns from the current function with `*kind`.
  **L1836 CN**: 以 `*kind` 从当前函数返回。
- **L1837 EN**: Transitions from the previous branch into the alternative path.
  **L1837 CN**: 从前一个分支过渡到备选路径。
- **L1838 EN**: Continues logic associated with callable symbol `die`.
  **L1838 CN**: 继续与可调用符号 `die` 相关的逻辑。
- **L1839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"INTERNAL: The __fortran_builtins module does not define the kind '%s'",`.
  **L1839 CN**: 继续一个多行参数列表、初始化器或聚合项：`"INTERNAL: The __fortran_builtins module does not define the kind '%s'",`。
- **L1840 EN**: Executes a standalone statement or declaration: `which);`.
  **L1840 CN**: 执行一条独立语句或声明：`which);`。
- **L1841 EN**: Returns from the current function with `-1`.
  **L1841 CN**: 以 `-1` 从当前函数返回。
- **L1842 EN**: Closes the current lexical scope or compound statement.
  **L1842 CN**: 结束当前词法作用域或复合语句块。
- **L1843 EN**: Closes the current lexical scope or compound statement.
  **L1843 CN**: 结束当前词法作用域或复合语句块。
- **L1844 EN**: Blank line separating nearby declarations or logic blocks.
  **L1844 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1845 EN**: Comment explains nearby logic, intent, or metadata: `Ensure that the keywords of arguments to MAX/MIN and their variants`.
  **L1845 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ensure that the keywords of arguments to MAX/MIN and their variants`。
- **L1846 EN**: Comment explains nearby logic, intent, or metadata: `are of the form A123 with no duplicates or leading zeroes.`.
  **L1846 CN**: 注释说明附近代码的逻辑、意图或元数据：`are of the form A123 with no duplicates or leading zeroes.`。
- **L1847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static bool CheckMaxMinArgument(parser::CharBlock keyword,`.
  **L1847 CN**: 继续一个多行参数列表、初始化器或聚合项：`static bool CheckMaxMinArgument(parser::CharBlock keyword,`。
- **L1848 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::set<parser::CharBlock> &set, const char *intrinsicName,`.
  **L1848 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::set<parser::CharBlock> &set, const char *intrinsicName,`。

### Lines 1849-1872

````cpp
    parser::ContextualMessages &messages) {
  std::size_t j{1};
  for (; j < keyword.size(); ++j) {
    char ch{(keyword)[j]};
    if (ch < (j == 1 ? '1' : '0') || ch > '9') {
      break;
    }
  }
  if (keyword.size() < 2 || (keyword)[0] != 'a' || j < keyword.size()) {
    messages.Say(keyword,
        "argument keyword '%s=' is not known in call to '%s'"_err_en_US,
        keyword, intrinsicName);
    return false;
  }
  if (!set.insert(keyword).second) {
    messages.Say(keyword,
        "argument keyword '%s=' was repeated in call to '%s'"_err_en_US,
        keyword, intrinsicName);
    return false;
  }
  return true;
}

// Validate the keyword, if any, and ensure that A1 and A2 are always placed in
````
- **L1849 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages &messages) {`.
  **L1849 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages &messages) {`。
- **L1850 EN**: Executes a standalone statement or declaration: `std::size_t j{1};`.
  **L1850 CN**: 执行一条独立语句或声明：`std::size_t j{1};`。
- **L1851 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1851 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1852 EN**: Executes a call or declaration centered on `ch{`.
  **L1852 CN**: 执行以 `ch{` 为核心的调用或声明。
- **L1853 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1853 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1854 EN**: Exits the nearest loop or switch statement.
  **L1854 CN**: 退出最近的循环或 switch 语句。
- **L1855 EN**: Closes the current lexical scope or compound statement.
  **L1855 CN**: 结束当前词法作用域或复合语句块。
- **L1856 EN**: Closes the current lexical scope or compound statement.
  **L1856 CN**: 结束当前词法作用域或复合语句块。
- **L1857 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1857 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(keyword,`.
  **L1858 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(keyword,`。
- **L1859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"argument keyword '%s=' is not known in call to '%s'"_err_en_US,`.
  **L1859 CN**: 继续一个多行参数列表、初始化器或聚合项：`"argument keyword '%s=' is not known in call to '%s'"_err_en_US,`。
- **L1860 EN**: Executes a standalone statement or declaration: `keyword, intrinsicName);`.
  **L1860 CN**: 执行一条独立语句或声明：`keyword, intrinsicName);`。
- **L1861 EN**: Returns from the current function with `false`.
  **L1861 CN**: 以 `false` 从当前函数返回。
- **L1862 EN**: Closes the current lexical scope or compound statement.
  **L1862 CN**: 结束当前词法作用域或复合语句块。
- **L1863 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1863 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(keyword,`.
  **L1864 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(keyword,`。
- **L1865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"argument keyword '%s=' was repeated in call to '%s'"_err_en_US,`.
  **L1865 CN**: 继续一个多行参数列表、初始化器或聚合项：`"argument keyword '%s=' was repeated in call to '%s'"_err_en_US,`。
- **L1866 EN**: Executes a standalone statement or declaration: `keyword, intrinsicName);`.
  **L1866 CN**: 执行一条独立语句或声明：`keyword, intrinsicName);`。
- **L1867 EN**: Returns from the current function with `false`.
  **L1867 CN**: 以 `false` 从当前函数返回。
- **L1868 EN**: Closes the current lexical scope or compound statement.
  **L1868 CN**: 结束当前词法作用域或复合语句块。
- **L1869 EN**: Returns from the current function with `true`.
  **L1869 CN**: 以 `true` 从当前函数返回。
- **L1870 EN**: Closes the current lexical scope or compound statement.
  **L1870 CN**: 结束当前词法作用域或复合语句块。
- **L1871 EN**: Blank line separating nearby declarations or logic blocks.
  **L1871 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1872 EN**: Comment explains nearby logic, intent, or metadata: `Validate the keyword, if any, and ensure that A1 and A2 are always placed in`.
  **L1872 CN**: 注释说明附近代码的逻辑、意图或元数据：`Validate the keyword, if any, and ensure that A1 and A2 are always placed in`。

### Lines 1873-1896

````cpp
// first and second position in actualForDummy. A1 and A2 are special since they
// are not optional. The rest of the arguments are not sorted, there are no
// differences between them.
static bool CheckAndPushMinMaxArgument(ActualArgument &arg,
    std::vector<ActualArgument *> &actualForDummy,
    std::set<parser::CharBlock> &set, const char *intrinsicName,
    parser::ContextualMessages &messages) {
  if (std::optional<parser::CharBlock> keyword{arg.keyword()}) {
    if (!CheckMaxMinArgument(*keyword, set, intrinsicName, messages)) {
      return false;
    }
    const bool isA1{*keyword == parser::CharBlock{"a1", 2}};
    if (isA1 && !actualForDummy[0]) {
      actualForDummy[0] = &arg;
      return true;
    }
    const bool isA2{*keyword == parser::CharBlock{"a2", 2}};
    if (isA2 && !actualForDummy[1]) {
      actualForDummy[1] = &arg;
      return true;
    }
    if (isA1 || isA2) {
      // Note that for arguments other than a1 and a2, this error will be caught
      // later in check-call.cpp.
````
- **L1873 EN**: Comment explains nearby logic, intent, or metadata: `first and second position in actualForDummy. A1 and A2 are special since they`.
  **L1873 CN**: 注释说明附近代码的逻辑、意图或元数据：`first and second position in actualForDummy. A1 and A2 are special since they`。
- **L1874 EN**: Comment explains nearby logic, intent, or metadata: `are not optional. The rest of the arguments are not sorted, there are no`.
  **L1874 CN**: 注释说明附近代码的逻辑、意图或元数据：`are not optional. The rest of the arguments are not sorted, there are no`。
- **L1875 EN**: Comment explains nearby logic, intent, or metadata: `differences between them.`.
  **L1875 CN**: 注释说明附近代码的逻辑、意图或元数据：`differences between them.`。
- **L1876 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static bool CheckAndPushMinMaxArgument(ActualArgument &arg,`.
  **L1876 CN**: 继续一个多行参数列表、初始化器或聚合项：`static bool CheckAndPushMinMaxArgument(ActualArgument &arg,`。
- **L1877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::vector<ActualArgument *> &actualForDummy,`.
  **L1877 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::vector<ActualArgument *> &actualForDummy,`。
- **L1878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::set<parser::CharBlock> &set, const char *intrinsicName,`.
  **L1878 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::set<parser::CharBlock> &set, const char *intrinsicName,`。
- **L1879 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages &messages) {`.
  **L1879 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages &messages) {`。
- **L1880 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1880 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1881 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1881 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1882 EN**: Returns from the current function with `false`.
  **L1882 CN**: 以 `false` 从当前函数返回。
- **L1883 EN**: Closes the current lexical scope or compound statement.
  **L1883 CN**: 结束当前词法作用域或复合语句块。
- **L1884 EN**: Executes a standalone statement or declaration: `const bool isA1{*keyword == parser::CharBlock{"a1", 2}};`.
  **L1884 CN**: 执行一条独立语句或声明：`const bool isA1{*keyword == parser::CharBlock{"a1", 2}};`。
- **L1885 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1885 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1886 EN**: Executes a standalone statement or declaration: `actualForDummy[0] = &arg;`.
  **L1886 CN**: 执行一条独立语句或声明：`actualForDummy[0] = &arg;`。
- **L1887 EN**: Returns from the current function with `true`.
  **L1887 CN**: 以 `true` 从当前函数返回。
- **L1888 EN**: Closes the current lexical scope or compound statement.
  **L1888 CN**: 结束当前词法作用域或复合语句块。
- **L1889 EN**: Executes a standalone statement or declaration: `const bool isA2{*keyword == parser::CharBlock{"a2", 2}};`.
  **L1889 CN**: 执行一条独立语句或声明：`const bool isA2{*keyword == parser::CharBlock{"a2", 2}};`。
- **L1890 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1890 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1891 EN**: Executes a standalone statement or declaration: `actualForDummy[1] = &arg;`.
  **L1891 CN**: 执行一条独立语句或声明：`actualForDummy[1] = &arg;`。
- **L1892 EN**: Returns from the current function with `true`.
  **L1892 CN**: 以 `true` 从当前函数返回。
- **L1893 EN**: Closes the current lexical scope or compound statement.
  **L1893 CN**: 结束当前词法作用域或复合语句块。
- **L1894 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1894 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1895 EN**: Comment explains nearby logic, intent, or metadata: `Note that for arguments other than a1 and a2, this error will be caught`.
  **L1895 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that for arguments other than a1 and a2, this error will be caught`。
- **L1896 EN**: Comment explains nearby logic, intent, or metadata: `later in check-call.cpp.`.
  **L1896 CN**: 注释说明附近代码的逻辑、意图或元数据：`later in check-call.cpp.`。

### Lines 1897-1920

````cpp
      messages.Say(*keyword,
          "keyword argument '%s=' to intrinsic '%s' was supplied "
          "positionally by an earlier actual argument"_err_en_US,
          *keyword, intrinsicName);
      return false;
    }
  } else {
    if (actualForDummy.size() == 2) {
      if (!actualForDummy[0] && !actualForDummy[1]) {
        actualForDummy[0] = &arg;
        return true;
      } else if (!actualForDummy[1]) {
        actualForDummy[1] = &arg;
        return true;
      }
    }
  }
  actualForDummy.push_back(&arg);
  return true;
}

static bool CheckAtomicKind(const ActualArgument &arg,
    const semantics::Scope *builtinsScope, parser::ContextualMessages &messages,
    const char *keyword) {
````
- **L1897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*keyword,`.
  **L1897 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*keyword,`。
- **L1898 EN**: Continues the surrounding expression or declaration: `"keyword argument '%s=' to intrinsic '%s' was supplied "`.
  **L1898 CN**: 继续构造周围的表达式或声明：`"keyword argument '%s=' to intrinsic '%s' was supplied "`。
- **L1899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"positionally by an earlier actual argument"_err_en_US,`.
  **L1899 CN**: 继续一个多行参数列表、初始化器或聚合项：`"positionally by an earlier actual argument"_err_en_US,`。
- **L1900 EN**: Comment explains nearby logic, intent, or metadata: `keyword, intrinsicName);`.
  **L1900 CN**: 注释说明附近代码的逻辑、意图或元数据：`keyword, intrinsicName);`。
- **L1901 EN**: Returns from the current function with `false`.
  **L1901 CN**: 以 `false` 从当前函数返回。
- **L1902 EN**: Closes the current lexical scope or compound statement.
  **L1902 CN**: 结束当前词法作用域或复合语句块。
- **L1903 EN**: Transitions from the previous branch into the alternative path.
  **L1903 CN**: 从前一个分支过渡到备选路径。
- **L1904 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1904 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1905 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1905 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1906 EN**: Executes a standalone statement or declaration: `actualForDummy[0] = &arg;`.
  **L1906 CN**: 执行一条独立语句或声明：`actualForDummy[0] = &arg;`。
- **L1907 EN**: Returns from the current function with `true`.
  **L1907 CN**: 以 `true` 从当前函数返回。
- **L1908 EN**: Transitions from the previous branch into an `else if` condition.
  **L1908 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1909 EN**: Executes a standalone statement or declaration: `actualForDummy[1] = &arg;`.
  **L1909 CN**: 执行一条独立语句或声明：`actualForDummy[1] = &arg;`。
- **L1910 EN**: Returns from the current function with `true`.
  **L1910 CN**: 以 `true` 从当前函数返回。
- **L1911 EN**: Closes the current lexical scope or compound statement.
  **L1911 CN**: 结束当前词法作用域或复合语句块。
- **L1912 EN**: Closes the current lexical scope or compound statement.
  **L1912 CN**: 结束当前词法作用域或复合语句块。
- **L1913 EN**: Closes the current lexical scope or compound statement.
  **L1913 CN**: 结束当前词法作用域或复合语句块。
- **L1914 EN**: Executes a call or declaration centered on `actualForDummy.push_back`.
  **L1914 CN**: 执行以 `actualForDummy.push_back` 为核心的调用或声明。
- **L1915 EN**: Returns from the current function with `true`.
  **L1915 CN**: 以 `true` 从当前函数返回。
- **L1916 EN**: Closes the current lexical scope or compound statement.
  **L1916 CN**: 结束当前词法作用域或复合语句块。
- **L1917 EN**: Blank line separating nearby declarations or logic blocks.
  **L1917 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static bool CheckAtomicKind(const ActualArgument &arg,`.
  **L1918 CN**: 继续一个多行参数列表、初始化器或聚合项：`static bool CheckAtomicKind(const ActualArgument &arg,`。
- **L1919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const semantics::Scope *builtinsScope, parser::ContextualMessages &messages,`.
  **L1919 CN**: 继续一个多行参数列表、初始化器或聚合项：`const semantics::Scope *builtinsScope, parser::ContextualMessages &messages,`。
- **L1920 EN**: Continues the surrounding expression or declaration: `const char *keyword) {`.
  **L1920 CN**: 继续构造周围的表达式或声明：`const char *keyword) {`。

### Lines 1921-1944

````cpp
  std::string atomicKindStr;
  std::optional<DynamicType> type{arg.GetType()};

  if (type->category() == TypeCategory::Integer) {
    atomicKindStr = "atomic_int_kind";
  } else if (type->category() == TypeCategory::Logical) {
    atomicKindStr = "atomic_logical_kind";
  } else {
    common::die("atomic_int_kind or atomic_logical_kind from iso_fortran_env "
                "must be used with IntType or LogicalType");
  }

  bool argOk{type->kind() ==
      GetBuiltinKind(builtinsScope, ("__builtin_" + atomicKindStr).c_str())};
  if (!argOk) {
    messages.Say(arg.sourceLocation(),
        "Actual argument for '%s=' must have kind=atomic_%s_kind, but is '%s'"_err_en_US,
        keyword, type->category() == TypeCategory::Integer ? "int" : "logical",
        type->AsFortran());
  }
  return argOk;
}

// Intrinsic interface matching against the arguments of a particular
````
- **L1921 EN**: Executes a standalone statement or declaration: `std::string atomicKindStr;`.
  **L1921 CN**: 执行一条独立语句或声明：`std::string atomicKindStr;`。
- **L1922 EN**: Executes a call or declaration centered on `type{arg.GetType`.
  **L1922 CN**: 执行以 `type{arg.GetType` 为核心的调用或声明。
- **L1923 EN**: Blank line separating nearby declarations or logic blocks.
  **L1923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1924 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1924 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1925 EN**: Executes a standalone statement or declaration: `atomicKindStr = "atomic_int_kind";`.
  **L1925 CN**: 执行一条独立语句或声明：`atomicKindStr = "atomic_int_kind";`。
- **L1926 EN**: Transitions from the previous branch into an `else if` condition.
  **L1926 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1927 EN**: Executes a standalone statement or declaration: `atomicKindStr = "atomic_logical_kind";`.
  **L1927 CN**: 执行一条独立语句或声明：`atomicKindStr = "atomic_logical_kind";`。
- **L1928 EN**: Transitions from the previous branch into the alternative path.
  **L1928 CN**: 从前一个分支过渡到备选路径。
- **L1929 EN**: Continues logic associated with callable symbol `die`.
  **L1929 CN**: 继续与可调用符号 `die` 相关的逻辑。
- **L1930 EN**: Executes a standalone statement or declaration: `"must be used with IntType or LogicalType");`.
  **L1930 CN**: 执行一条独立语句或声明：`"must be used with IntType or LogicalType");`。
- **L1931 EN**: Closes the current lexical scope or compound statement.
  **L1931 CN**: 结束当前词法作用域或复合语句块。
- **L1932 EN**: Blank line separating nearby declarations or logic blocks.
  **L1932 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1933 EN**: Continues logic associated with callable symbol `kind`.
  **L1933 CN**: 继续与可调用符号 `kind` 相关的逻辑。
- **L1934 EN**: Executes a call or declaration centered on `GetBuiltinKind`.
  **L1934 CN**: 执行以 `GetBuiltinKind` 为核心的调用或声明。
- **L1935 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1935 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg.sourceLocation(),`.
  **L1936 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg.sourceLocation(),`。
- **L1937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument for '%s=' must have kind=atomic_%s_kind, but is '%s'"_err_en_US,`.
  **L1937 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument for '%s=' must have kind=atomic_%s_kind, but is '%s'"_err_en_US,`。
- **L1938 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `keyword, type->category() == TypeCategory::Integer ? "int" : "logical",`.
  **L1938 CN**: 继续一个多行参数列表、初始化器或聚合项：`keyword, type->category() == TypeCategory::Integer ? "int" : "logical",`。
- **L1939 EN**: Executes a call or declaration centered on `type->AsFortran`.
  **L1939 CN**: 执行以 `type->AsFortran` 为核心的调用或声明。
- **L1940 EN**: Closes the current lexical scope or compound statement.
  **L1940 CN**: 结束当前词法作用域或复合语句块。
- **L1941 EN**: Returns from the current function with `argOk`.
  **L1941 CN**: 以 `argOk` 从当前函数返回。
- **L1942 EN**: Closes the current lexical scope or compound statement.
  **L1942 CN**: 结束当前词法作用域或复合语句块。
- **L1943 EN**: Blank line separating nearby declarations or logic blocks.
  **L1943 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1944 EN**: Comment explains nearby logic, intent, or metadata: `Intrinsic interface matching against the arguments of a particular`.
  **L1944 CN**: 注释说明附近代码的逻辑、意图或元数据：`Intrinsic interface matching against the arguments of a particular`。

### Lines 1945-1968

````cpp
// procedure reference.
std::optional<SpecificCall> IntrinsicInterface::Match(
    const CallCharacteristics &call,
    const common::IntrinsicTypeDefaultKinds &defaults,
    ActualArguments &arguments, FoldingContext &context,
    const semantics::Scope *builtinsScope) const {
  auto &messages{context.messages()};
  // Attempt to construct a 1-1 correspondence between the dummy arguments in
  // a particular intrinsic procedure's generic interface and the actual
  // arguments in a procedure reference.
  std::size_t dummyArgPatterns{0};
  for (; dummyArgPatterns < maxArguments && dummy[dummyArgPatterns].keyword;
       ++dummyArgPatterns) {
  }
  // MAX and MIN (and others that map to them) allow their last argument to
  // be repeated indefinitely.  The actualForDummy vector is sized
  // and null-initialized to the non-repeated dummy argument count
  // for other intrinsics.
  bool isMaxMin{dummyArgPatterns > 0 &&
      dummy[dummyArgPatterns - 1].optionality == Optionality::repeats};
  std::vector<ActualArgument *> actualForDummy(
      isMaxMin ? 2 : dummyArgPatterns, nullptr);
  bool anyMissingActualArgument{false};
  std::set<parser::CharBlock> maxMinKeywords;
````
- **L1945 EN**: Comment explains nearby logic, intent, or metadata: `procedure reference.`.
  **L1945 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure reference.`。
- **L1946 EN**: Continues logic associated with callable symbol `Match`.
  **L1946 CN**: 继续与可调用符号 `Match` 相关的逻辑。
- **L1947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const CallCharacteristics &call,`.
  **L1947 CN**: 继续一个多行参数列表、初始化器或聚合项：`const CallCharacteristics &call,`。
- **L1948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const common::IntrinsicTypeDefaultKinds &defaults,`.
  **L1948 CN**: 继续一个多行参数列表、初始化器或聚合项：`const common::IntrinsicTypeDefaultKinds &defaults,`。
- **L1949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ActualArguments &arguments, FoldingContext &context,`.
  **L1949 CN**: 继续一个多行参数列表、初始化器或聚合项：`ActualArguments &arguments, FoldingContext &context,`。
- **L1950 EN**: Continues the surrounding expression or declaration: `const semantics::Scope *builtinsScope) const {`.
  **L1950 CN**: 继续构造周围的表达式或声明：`const semantics::Scope *builtinsScope) const {`。
- **L1951 EN**: Executes a call or declaration centered on `&messages{context.messages`.
  **L1951 CN**: 执行以 `&messages{context.messages` 为核心的调用或声明。
- **L1952 EN**: Comment explains nearby logic, intent, or metadata: `Attempt to construct a 1-1 correspondence between the dummy arguments in`.
  **L1952 CN**: 注释说明附近代码的逻辑、意图或元数据：`Attempt to construct a 1-1 correspondence between the dummy arguments in`。
- **L1953 EN**: Comment explains nearby logic, intent, or metadata: `a particular intrinsic procedure's generic interface and the actual`.
  **L1953 CN**: 注释说明附近代码的逻辑、意图或元数据：`a particular intrinsic procedure's generic interface and the actual`。
- **L1954 EN**: Comment explains nearby logic, intent, or metadata: `arguments in a procedure reference.`.
  **L1954 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments in a procedure reference.`。
- **L1955 EN**: Executes a standalone statement or declaration: `std::size_t dummyArgPatterns{0};`.
  **L1955 CN**: 执行一条独立语句或声明：`std::size_t dummyArgPatterns{0};`。
- **L1956 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1956 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1957 EN**: Continues the surrounding expression or declaration: `++dummyArgPatterns) {`.
  **L1957 CN**: 继续构造周围的表达式或声明：`++dummyArgPatterns) {`。
- **L1958 EN**: Closes the current lexical scope or compound statement.
  **L1958 CN**: 结束当前词法作用域或复合语句块。
- **L1959 EN**: Comment explains nearby logic, intent, or metadata: `MAX and MIN (and others that map to them) allow their last argument to`.
  **L1959 CN**: 注释说明附近代码的逻辑、意图或元数据：`MAX and MIN (and others that map to them) allow their last argument to`。
- **L1960 EN**: Comment explains nearby logic, intent, or metadata: `be repeated indefinitely.  The actualForDummy vector is sized`.
  **L1960 CN**: 注释说明附近代码的逻辑、意图或元数据：`be repeated indefinitely.  The actualForDummy vector is sized`。
- **L1961 EN**: Comment explains nearby logic, intent, or metadata: `and null-initialized to the non-repeated dummy argument count`.
  **L1961 CN**: 注释说明附近代码的逻辑、意图或元数据：`and null-initialized to the non-repeated dummy argument count`。
- **L1962 EN**: Comment explains nearby logic, intent, or metadata: `for other intrinsics.`.
  **L1962 CN**: 注释说明附近代码的逻辑、意图或元数据：`for other intrinsics.`。
- **L1963 EN**: Continues the surrounding expression or declaration: `bool isMaxMin{dummyArgPatterns > 0 &&`.
  **L1963 CN**: 继续构造周围的表达式或声明：`bool isMaxMin{dummyArgPatterns > 0 &&`。
- **L1964 EN**: Executes a standalone statement or declaration: `dummy[dummyArgPatterns - 1].optionality == Optionality::repeats};`.
  **L1964 CN**: 执行一条独立语句或声明：`dummy[dummyArgPatterns - 1].optionality == Optionality::repeats};`。
- **L1965 EN**: Continues logic associated with callable symbol `actualForDummy`.
  **L1965 CN**: 继续与可调用符号 `actualForDummy` 相关的逻辑。
- **L1966 EN**: Executes a standalone statement or declaration: `isMaxMin ? 2 : dummyArgPatterns, nullptr);`.
  **L1966 CN**: 执行一条独立语句或声明：`isMaxMin ? 2 : dummyArgPatterns, nullptr);`。
- **L1967 EN**: Executes a standalone statement or declaration: `bool anyMissingActualArgument{false};`.
  **L1967 CN**: 执行一条独立语句或声明：`bool anyMissingActualArgument{false};`。
- **L1968 EN**: Executes a standalone statement or declaration: `std::set<parser::CharBlock> maxMinKeywords;`.
  **L1968 CN**: 执行一条独立语句或声明：`std::set<parser::CharBlock> maxMinKeywords;`。

### Lines 1969-1992

````cpp
  bool anyKeyword{false};
  int which{0};
  for (std::optional<ActualArgument> &arg : arguments) {
    ++which;
    if (arg) {
      if (arg->isAlternateReturn()) {
        messages.Say(arg->sourceLocation(),
            "alternate return specifier not acceptable on call to intrinsic '%s'"_err_en_US,
            name);
        return std::nullopt;
      }
      if (arg->keyword()) {
        anyKeyword = true;
      } else if (anyKeyword) {
        messages.Say(arg ? arg->sourceLocation() : std::nullopt,
            "actual argument #%d without a keyword may not follow an actual argument with a keyword"_err_en_US,
            which);
        return std::nullopt;
      }
    } else {
      anyMissingActualArgument = true;
      continue;
    }
    if (isMaxMin) {
````
- **L1969 EN**: Executes a standalone statement or declaration: `bool anyKeyword{false};`.
  **L1969 CN**: 执行一条独立语句或声明：`bool anyKeyword{false};`。
- **L1970 EN**: Executes a standalone statement or declaration: `int which{0};`.
  **L1970 CN**: 执行一条独立语句或声明：`int which{0};`。
- **L1971 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1971 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1972 EN**: Executes a standalone statement or declaration: `++which;`.
  **L1972 CN**: 执行一条独立语句或声明：`++which;`。
- **L1973 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1973 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1974 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1974 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L1975 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L1976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"alternate return specifier not acceptable on call to intrinsic '%s'"_err_en_US,`.
  **L1976 CN**: 继续一个多行参数列表、初始化器或聚合项：`"alternate return specifier not acceptable on call to intrinsic '%s'"_err_en_US,`。
- **L1977 EN**: Executes a standalone statement or declaration: `name);`.
  **L1977 CN**: 执行一条独立语句或声明：`name);`。
- **L1978 EN**: Returns from the current function with `std::nullopt`.
  **L1978 CN**: 以 `std::nullopt` 从当前函数返回。
- **L1979 EN**: Closes the current lexical scope or compound statement.
  **L1979 CN**: 结束当前词法作用域或复合语句块。
- **L1980 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1980 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1981 EN**: Executes a standalone statement or declaration: `anyKeyword = true;`.
  **L1981 CN**: 执行一条独立语句或声明：`anyKeyword = true;`。
- **L1982 EN**: Transitions from the previous branch into an `else if` condition.
  **L1982 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg ? arg->sourceLocation() : std::nullopt,`.
  **L1983 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg ? arg->sourceLocation() : std::nullopt,`。
- **L1984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"actual argument #%d without a keyword may not follow an actual argument with a keyword"_err_en_US,`.
  **L1984 CN**: 继续一个多行参数列表、初始化器或聚合项：`"actual argument #%d without a keyword may not follow an actual argument with a keyword"_err_en_US,`。
- **L1985 EN**: Executes a standalone statement or declaration: `which);`.
  **L1985 CN**: 执行一条独立语句或声明：`which);`。
- **L1986 EN**: Returns from the current function with `std::nullopt`.
  **L1986 CN**: 以 `std::nullopt` 从当前函数返回。
- **L1987 EN**: Closes the current lexical scope or compound statement.
  **L1987 CN**: 结束当前词法作用域或复合语句块。
- **L1988 EN**: Transitions from the previous branch into the alternative path.
  **L1988 CN**: 从前一个分支过渡到备选路径。
- **L1989 EN**: Executes a standalone statement or declaration: `anyMissingActualArgument = true;`.
  **L1989 CN**: 执行一条独立语句或声明：`anyMissingActualArgument = true;`。
- **L1990 EN**: Skips to the next loop iteration.
  **L1990 CN**: 跳到下一次循环迭代。
- **L1991 EN**: Closes the current lexical scope or compound statement.
  **L1991 CN**: 结束当前词法作用域或复合语句块。
- **L1992 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1992 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1993-2016

````cpp
      if (!CheckAndPushMinMaxArgument(
              *arg, actualForDummy, maxMinKeywords, name, messages)) {
        return std::nullopt;
      }
    } else {
      bool found{false};
      for (std::size_t j{0}; j < dummyArgPatterns && !found; ++j) {
        if (dummy[j].optionality == Optionality::missing) {
          continue;
        }
        if (arg->keyword()) {
          found = *arg->keyword() == dummy[j].keyword;
          if (found) {
            if (const auto *previous{actualForDummy[j]}) {
              if (previous->keyword()) {
                messages.Say(*arg->keyword(),
                    "repeated keyword argument to intrinsic '%s'"_err_en_US,
                    name);
              } else {
                messages.Say(*arg->keyword(),
                    "keyword argument to intrinsic '%s' was supplied "
                    "positionally by an earlier actual argument"_err_en_US,
                    name);
              }
````
- **L1993 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1993 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1994 EN**: Comment explains nearby logic, intent, or metadata: `arg, actualForDummy, maxMinKeywords, name, messages)) {`.
  **L1994 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg, actualForDummy, maxMinKeywords, name, messages)) {`。
- **L1995 EN**: Returns from the current function with `std::nullopt`.
  **L1995 CN**: 以 `std::nullopt` 从当前函数返回。
- **L1996 EN**: Closes the current lexical scope or compound statement.
  **L1996 CN**: 结束当前词法作用域或复合语句块。
- **L1997 EN**: Transitions from the previous branch into the alternative path.
  **L1997 CN**: 从前一个分支过渡到备选路径。
- **L1998 EN**: Executes a standalone statement or declaration: `bool found{false};`.
  **L1998 CN**: 执行一条独立语句或声明：`bool found{false};`。
- **L1999 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1999 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2000 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2000 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2001 EN**: Skips to the next loop iteration.
  **L2001 CN**: 跳到下一次循环迭代。
- **L2002 EN**: Closes the current lexical scope or compound statement.
  **L2002 CN**: 结束当前词法作用域或复合语句块。
- **L2003 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2003 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2004 EN**: Executes a call or declaration centered on `*arg->keyword`.
  **L2004 CN**: 执行以 `*arg->keyword` 为核心的调用或声明。
- **L2005 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2005 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2006 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2006 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2007 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2007 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*arg->keyword(),`.
  **L2008 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*arg->keyword(),`。
- **L2009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"repeated keyword argument to intrinsic '%s'"_err_en_US,`.
  **L2009 CN**: 继续一个多行参数列表、初始化器或聚合项：`"repeated keyword argument to intrinsic '%s'"_err_en_US,`。
- **L2010 EN**: Executes a standalone statement or declaration: `name);`.
  **L2010 CN**: 执行一条独立语句或声明：`name);`。
- **L2011 EN**: Transitions from the previous branch into the alternative path.
  **L2011 CN**: 从前一个分支过渡到备选路径。
- **L2012 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*arg->keyword(),`.
  **L2012 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*arg->keyword(),`。
- **L2013 EN**: Continues the surrounding expression or declaration: `"keyword argument to intrinsic '%s' was supplied "`.
  **L2013 CN**: 继续构造周围的表达式或声明：`"keyword argument to intrinsic '%s' was supplied "`。
- **L2014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"positionally by an earlier actual argument"_err_en_US,`.
  **L2014 CN**: 继续一个多行参数列表、初始化器或聚合项：`"positionally by an earlier actual argument"_err_en_US,`。
- **L2015 EN**: Executes a standalone statement or declaration: `name);`.
  **L2015 CN**: 执行一条独立语句或声明：`name);`。
- **L2016 EN**: Closes the current lexical scope or compound statement.
  **L2016 CN**: 结束当前词法作用域或复合语句块。

### Lines 2017-2040

````cpp
              return std::nullopt;
            }
          }
        } else {
          found = !actualForDummy[j] && !anyMissingActualArgument;
        }
        if (found) {
          actualForDummy[j] = &*arg;
        }
      }
      if (!found) {
        if (arg->keyword()) {
          messages.Say(*arg->keyword(),
              "unknown keyword argument to intrinsic '%s'"_err_en_US, name);
        } else {
          messages.Say(
              "too many actual arguments for intrinsic '%s'"_err_en_US, name);
        }
        return std::nullopt;
      }
    }
  }

  std::size_t dummies{actualForDummy.size()};
````
- **L2017 EN**: Returns from the current function with `std::nullopt`.
  **L2017 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2018 EN**: Closes the current lexical scope or compound statement.
  **L2018 CN**: 结束当前词法作用域或复合语句块。
- **L2019 EN**: Closes the current lexical scope or compound statement.
  **L2019 CN**: 结束当前词法作用域或复合语句块。
- **L2020 EN**: Transitions from the previous branch into the alternative path.
  **L2020 CN**: 从前一个分支过渡到备选路径。
- **L2021 EN**: Executes a standalone statement or declaration: `found = !actualForDummy[j] && !anyMissingActualArgument;`.
  **L2021 CN**: 执行一条独立语句或声明：`found = !actualForDummy[j] && !anyMissingActualArgument;`。
- **L2022 EN**: Closes the current lexical scope or compound statement.
  **L2022 CN**: 结束当前词法作用域或复合语句块。
- **L2023 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2023 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2024 EN**: Executes a standalone statement or declaration: `actualForDummy[j] = &*arg;`.
  **L2024 CN**: 执行一条独立语句或声明：`actualForDummy[j] = &*arg;`。
- **L2025 EN**: Closes the current lexical scope or compound statement.
  **L2025 CN**: 结束当前词法作用域或复合语句块。
- **L2026 EN**: Closes the current lexical scope or compound statement.
  **L2026 CN**: 结束当前词法作用域或复合语句块。
- **L2027 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2027 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2028 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2028 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*arg->keyword(),`.
  **L2029 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*arg->keyword(),`。
- **L2030 EN**: Executes a standalone statement or declaration: `"unknown keyword argument to intrinsic '%s'"_err_en_US, name);`.
  **L2030 CN**: 执行一条独立语句或声明：`"unknown keyword argument to intrinsic '%s'"_err_en_US, name);`。
- **L2031 EN**: Transitions from the previous branch into the alternative path.
  **L2031 CN**: 从前一个分支过渡到备选路径。
- **L2032 EN**: Continues logic associated with callable symbol `Say`.
  **L2032 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2033 EN**: Executes a standalone statement or declaration: `"too many actual arguments for intrinsic '%s'"_err_en_US, name);`.
  **L2033 CN**: 执行一条独立语句或声明：`"too many actual arguments for intrinsic '%s'"_err_en_US, name);`。
- **L2034 EN**: Closes the current lexical scope or compound statement.
  **L2034 CN**: 结束当前词法作用域或复合语句块。
- **L2035 EN**: Returns from the current function with `std::nullopt`.
  **L2035 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2036 EN**: Closes the current lexical scope or compound statement.
  **L2036 CN**: 结束当前词法作用域或复合语句块。
- **L2037 EN**: Closes the current lexical scope or compound statement.
  **L2037 CN**: 结束当前词法作用域或复合语句块。
- **L2038 EN**: Closes the current lexical scope or compound statement.
  **L2038 CN**: 结束当前词法作用域或复合语句块。
- **L2039 EN**: Blank line separating nearby declarations or logic blocks.
  **L2039 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2040 EN**: Executes a call or declaration centered on `dummies{actualForDummy.size`.
  **L2040 CN**: 执行以 `dummies{actualForDummy.size` 为核心的调用或声明。

### Lines 2041-2064

````cpp

  // Check types and kinds of the actual arguments against the intrinsic's
  // interface.  Ensure that two or more arguments that have to have the same
  // (or compatible) type and kind do so.  Check for missing non-optional
  // arguments now, too.
  const ActualArgument *sameArg{nullptr};
  const ActualArgument *operandArg{nullptr};
  const IntrinsicDummyArgument *kindDummyArg{nullptr};
  const ActualArgument *kindArg{nullptr};
  std::optional<int> dimArg;
  for (std::size_t j{0}; j < dummies; ++j) {
    const IntrinsicDummyArgument &d{dummy[std::min(j, dummyArgPatterns - 1)]};
    if (d.typePattern.kindCode == KindCode::kindArg) {
      CHECK(!kindDummyArg);
      kindDummyArg = &d;
    }
    const ActualArgument *arg{actualForDummy[j]};
    if (!arg) {
      if (d.optionality == Optionality::required) {
        std::string kw{d.keyword};
        if (isMaxMin && !actualForDummy[0] && !actualForDummy[1]) {
          messages.Say("missing mandatory 'a1=' and 'a2=' arguments"_err_en_US);
        } else {
          messages.Say(
````
- **L2041 EN**: Blank line separating nearby declarations or logic blocks.
  **L2041 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2042 EN**: Comment explains nearby logic, intent, or metadata: `Check types and kinds of the actual arguments against the intrinsic's`.
  **L2042 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check types and kinds of the actual arguments against the intrinsic's`。
- **L2043 EN**: Comment explains nearby logic, intent, or metadata: `interface.  Ensure that two or more arguments that have to have the same`.
  **L2043 CN**: 注释说明附近代码的逻辑、意图或元数据：`interface.  Ensure that two or more arguments that have to have the same`。
- **L2044 EN**: Comment explains nearby logic, intent, or metadata: `(or compatible) type and kind do so.  Check for missing non-optional`.
  **L2044 CN**: 注释说明附近代码的逻辑、意图或元数据：`(or compatible) type and kind do so.  Check for missing non-optional`。
- **L2045 EN**: Comment explains nearby logic, intent, or metadata: `arguments now, too.`.
  **L2045 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments now, too.`。
- **L2046 EN**: Executes a standalone statement or declaration: `const ActualArgument *sameArg{nullptr};`.
  **L2046 CN**: 执行一条独立语句或声明：`const ActualArgument *sameArg{nullptr};`。
- **L2047 EN**: Executes a standalone statement or declaration: `const ActualArgument *operandArg{nullptr};`.
  **L2047 CN**: 执行一条独立语句或声明：`const ActualArgument *operandArg{nullptr};`。
- **L2048 EN**: Executes a standalone statement or declaration: `const IntrinsicDummyArgument *kindDummyArg{nullptr};`.
  **L2048 CN**: 执行一条独立语句或声明：`const IntrinsicDummyArgument *kindDummyArg{nullptr};`。
- **L2049 EN**: Executes a standalone statement or declaration: `const ActualArgument *kindArg{nullptr};`.
  **L2049 CN**: 执行一条独立语句或声明：`const ActualArgument *kindArg{nullptr};`。
- **L2050 EN**: Executes a standalone statement or declaration: `std::optional<int> dimArg;`.
  **L2050 CN**: 执行一条独立语句或声明：`std::optional<int> dimArg;`。
- **L2051 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2051 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2052 EN**: Executes a call or declaration centered on `&d{dummy[std::min`.
  **L2052 CN**: 执行以 `&d{dummy[std::min` 为核心的调用或声明。
- **L2053 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2053 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2054 EN**: Executes a call or declaration centered on `CHECK`.
  **L2054 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2055 EN**: Executes a standalone statement or declaration: `kindDummyArg = &d;`.
  **L2055 CN**: 执行一条独立语句或声明：`kindDummyArg = &d;`。
- **L2056 EN**: Closes the current lexical scope or compound statement.
  **L2056 CN**: 结束当前词法作用域或复合语句块。
- **L2057 EN**: Executes a standalone statement or declaration: `const ActualArgument *arg{actualForDummy[j]};`.
  **L2057 CN**: 执行一条独立语句或声明：`const ActualArgument *arg{actualForDummy[j]};`。
- **L2058 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2058 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2059 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2059 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2060 EN**: Executes a standalone statement or declaration: `std::string kw{d.keyword};`.
  **L2060 CN**: 执行一条独立语句或声明：`std::string kw{d.keyword};`。
- **L2061 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2061 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2062 EN**: Executes a call or declaration centered on `messages.Say`.
  **L2062 CN**: 执行以 `messages.Say` 为核心的调用或声明。
- **L2063 EN**: Transitions from the previous branch into the alternative path.
  **L2063 CN**: 从前一个分支过渡到备选路径。
- **L2064 EN**: Continues logic associated with callable symbol `Say`.
  **L2064 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 2065-2088

````cpp
              "missing mandatory '%s=' argument"_err_en_US, kw.c_str());
        }
        return std::nullopt; // missing non-OPTIONAL argument
      } else {
        continue;
      }
    }
    if (d.optionality == Optionality::missing) {
      messages.Say(arg->sourceLocation(), "unexpected '%s=' argument"_err_en_US,
          d.keyword);
      return std::nullopt;
    }
    if (!d.flags.test(ArgFlag::canBeNullPointer)) {
      if (const auto *expr{arg->UnwrapExpr()}; IsNullPointer(expr)) {
        if (!IsBareNullPointer(expr) && IsNullObjectPointer(expr) &&
            d.flags.test(ArgFlag::canBeMoldNull)) {
          // ok
        } else {
          messages.Say(arg->sourceLocation(),
              "A NULL() pointer is not allowed for '%s=' intrinsic argument"_err_en_US,
              d.keyword);
          return std::nullopt;
        }
      }
````
- **L2065 EN**: Executes a call or declaration centered on `kw.c_str`.
  **L2065 CN**: 执行以 `kw.c_str` 为核心的调用或声明。
- **L2066 EN**: Closes the current lexical scope or compound statement.
  **L2066 CN**: 结束当前词法作用域或复合语句块。
- **L2067 EN**: Returns from the current function with `std::nullopt; // missing non-OPTIONAL argument`.
  **L2067 CN**: 以 `std::nullopt; // missing non-OPTIONAL argument` 从当前函数返回。
- **L2068 EN**: Transitions from the previous branch into the alternative path.
  **L2068 CN**: 从前一个分支过渡到备选路径。
- **L2069 EN**: Skips to the next loop iteration.
  **L2069 CN**: 跳到下一次循环迭代。
- **L2070 EN**: Closes the current lexical scope or compound statement.
  **L2070 CN**: 结束当前词法作用域或复合语句块。
- **L2071 EN**: Closes the current lexical scope or compound statement.
  **L2071 CN**: 结束当前词法作用域或复合语句块。
- **L2072 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2072 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(), "unexpected '%s=' argument"_err_en_US,`.
  **L2073 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(), "unexpected '%s=' argument"_err_en_US,`。
- **L2074 EN**: Executes a standalone statement or declaration: `d.keyword);`.
  **L2074 CN**: 执行一条独立语句或声明：`d.keyword);`。
- **L2075 EN**: Returns from the current function with `std::nullopt`.
  **L2075 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2076 EN**: Closes the current lexical scope or compound statement.
  **L2076 CN**: 结束当前词法作用域或复合语句块。
- **L2077 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2077 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2078 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2078 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2079 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2079 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2080 EN**: Starts a function, method, lambda, or structured scope: `d.flags.test(ArgFlag::canBeMoldNull)) {`.
  **L2080 CN**: 开始一个函数、方法、lambda 或结构化作用域：`d.flags.test(ArgFlag::canBeMoldNull)) {`。
- **L2081 EN**: Comment explains nearby logic, intent, or metadata: `ok`.
  **L2081 CN**: 注释说明附近代码的逻辑、意图或元数据：`ok`。
- **L2082 EN**: Transitions from the previous branch into the alternative path.
  **L2082 CN**: 从前一个分支过渡到备选路径。
- **L2083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2083 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"A NULL() pointer is not allowed for '%s=' intrinsic argument"_err_en_US,`.
  **L2084 CN**: 继续一个多行参数列表、初始化器或聚合项：`"A NULL() pointer is not allowed for '%s=' intrinsic argument"_err_en_US,`。
- **L2085 EN**: Executes a standalone statement or declaration: `d.keyword);`.
  **L2085 CN**: 执行一条独立语句或声明：`d.keyword);`。
- **L2086 EN**: Returns from the current function with `std::nullopt`.
  **L2086 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2087 EN**: Closes the current lexical scope or compound statement.
  **L2087 CN**: 结束当前词法作用域或复合语句块。
- **L2088 EN**: Closes the current lexical scope or compound statement.
  **L2088 CN**: 结束当前词法作用域或复合语句块。

### Lines 2089-2112

````cpp
    }
    if (!d.flags.test(ArgFlag::canBeNullAllocatable) &&
        IsNullAllocatable(arg->UnwrapExpr()) &&
        !d.flags.test(ArgFlag::canBeMoldNull)) {
      messages.Say(arg->sourceLocation(),
          "A NULL() allocatable is not allowed for '%s=' intrinsic argument"_err_en_US,
          d.keyword);
      return std::nullopt;
    }
    if (d.flags.test(ArgFlag::notAssumedSize)) {
      if (auto named{ExtractNamedEntity(*arg)}) {
        if (semantics::IsAssumedSizeArray(named->GetLastSymbol())) {
          messages.Say(arg->sourceLocation(),
              "The '%s=' argument to the intrinsic procedure '%s' may not be assumed-size"_err_en_US,
              d.keyword, name);
          return std::nullopt;
        }
      }
    }
    if (arg->GetAssumedTypeDummy()) {
      // TYPE(*) assumed-type dummy argument forwarded to intrinsic
      if (d.typePattern.categorySet == AnyType &&
          (d.rank == Rank::anyOrAssumedRank ||
              d.rank == Rank::arrayOrAssumedRank) &&
````
- **L2089 EN**: Closes the current lexical scope or compound statement.
  **L2089 CN**: 结束当前词法作用域或复合语句块。
- **L2090 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2090 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2091 EN**: Continues logic associated with callable symbol `IsNullAllocatable`.
  **L2091 CN**: 继续与可调用符号 `IsNullAllocatable` 相关的逻辑。
- **L2092 EN**: Starts a function, method, lambda, or structured scope: `!d.flags.test(ArgFlag::canBeMoldNull)) {`.
  **L2092 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!d.flags.test(ArgFlag::canBeMoldNull)) {`。
- **L2093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2093 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"A NULL() allocatable is not allowed for '%s=' intrinsic argument"_err_en_US,`.
  **L2094 CN**: 继续一个多行参数列表、初始化器或聚合项：`"A NULL() allocatable is not allowed for '%s=' intrinsic argument"_err_en_US,`。
- **L2095 EN**: Executes a standalone statement or declaration: `d.keyword);`.
  **L2095 CN**: 执行一条独立语句或声明：`d.keyword);`。
- **L2096 EN**: Returns from the current function with `std::nullopt`.
  **L2096 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2097 EN**: Closes the current lexical scope or compound statement.
  **L2097 CN**: 结束当前词法作用域或复合语句块。
- **L2098 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2098 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2099 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2099 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2100 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2100 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2101 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The '%s=' argument to the intrinsic procedure '%s' may not be assumed-size"_err_en_US,`.
  **L2102 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The '%s=' argument to the intrinsic procedure '%s' may not be assumed-size"_err_en_US,`。
- **L2103 EN**: Executes a standalone statement or declaration: `d.keyword, name);`.
  **L2103 CN**: 执行一条独立语句或声明：`d.keyword, name);`。
- **L2104 EN**: Returns from the current function with `std::nullopt`.
  **L2104 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2105 EN**: Closes the current lexical scope or compound statement.
  **L2105 CN**: 结束当前词法作用域或复合语句块。
- **L2106 EN**: Closes the current lexical scope or compound statement.
  **L2106 CN**: 结束当前词法作用域或复合语句块。
- **L2107 EN**: Closes the current lexical scope or compound statement.
  **L2107 CN**: 结束当前词法作用域或复合语句块。
- **L2108 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2108 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2109 EN**: Comment explains nearby logic, intent, or metadata: `TYPE(*) assumed-type dummy argument forwarded to intrinsic`.
  **L2109 CN**: 注释说明附近代码的逻辑、意图或元数据：`TYPE(*) assumed-type dummy argument forwarded to intrinsic`。
- **L2110 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2110 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2111 EN**: Continues the surrounding expression or declaration: `(d.rank == Rank::anyOrAssumedRank ||`.
  **L2111 CN**: 继续构造周围的表达式或声明：`(d.rank == Rank::anyOrAssumedRank ||`。
- **L2112 EN**: Continues the surrounding expression or declaration: `d.rank == Rank::arrayOrAssumedRank) &&`.
  **L2112 CN**: 继续构造周围的表达式或声明：`d.rank == Rank::arrayOrAssumedRank) &&`。

### Lines 2113-2136

````cpp
          (d.typePattern.kindCode == KindCode::any ||
              d.typePattern.kindCode == KindCode::addressable)) {
        continue;
      } else {
        messages.Say(arg->sourceLocation(),
            "Assumed type TYPE(*) dummy argument not allowed for '%s=' intrinsic argument"_err_en_US,
            d.keyword);
        return std::nullopt;
      }
    }
    std::optional<DynamicType> type{arg->GetType()};
    if (!type) {
      CHECK(arg->Rank() == 0);
      const Expr<SomeType> &expr{DEREF(arg->UnwrapExpr())};
      if (IsBOZLiteral(expr)) {
        if (d.typePattern.kindCode == KindCode::typeless ||
            d.rank == Rank::elementalOrBOZ) {
          continue;
        } else {
          const IntrinsicDummyArgument *nextParam{
              j + 1 < dummies ? &dummy[j + 1] : nullptr};
          if (nextParam && nextParam->rank == Rank::elementalOrBOZ) {
            messages.Say(arg->sourceLocation(),
                "Typeless (BOZ) not allowed for both '%s=' & '%s=' arguments"_err_en_US, // C7109
````
- **L2113 EN**: Continues the surrounding expression or declaration: `(d.typePattern.kindCode == KindCode::any ||`.
  **L2113 CN**: 继续构造周围的表达式或声明：`(d.typePattern.kindCode == KindCode::any ||`。
- **L2114 EN**: Continues the surrounding expression or declaration: `d.typePattern.kindCode == KindCode::addressable)) {`.
  **L2114 CN**: 继续构造周围的表达式或声明：`d.typePattern.kindCode == KindCode::addressable)) {`。
- **L2115 EN**: Skips to the next loop iteration.
  **L2115 CN**: 跳到下一次循环迭代。
- **L2116 EN**: Transitions from the previous branch into the alternative path.
  **L2116 CN**: 从前一个分支过渡到备选路径。
- **L2117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2117 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed type TYPE(*) dummy argument not allowed for '%s=' intrinsic argument"_err_en_US,`.
  **L2118 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed type TYPE(*) dummy argument not allowed for '%s=' intrinsic argument"_err_en_US,`。
- **L2119 EN**: Executes a standalone statement or declaration: `d.keyword);`.
  **L2119 CN**: 执行一条独立语句或声明：`d.keyword);`。
- **L2120 EN**: Returns from the current function with `std::nullopt`.
  **L2120 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2121 EN**: Closes the current lexical scope or compound statement.
  **L2121 CN**: 结束当前词法作用域或复合语句块。
- **L2122 EN**: Closes the current lexical scope or compound statement.
  **L2122 CN**: 结束当前词法作用域或复合语句块。
- **L2123 EN**: Executes a call or declaration centered on `type{arg->GetType`.
  **L2123 CN**: 执行以 `type{arg->GetType` 为核心的调用或声明。
- **L2124 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2124 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2125 EN**: Executes a call or declaration centered on `CHECK`.
  **L2125 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2126 EN**: Executes a call or declaration centered on `&expr{DEREF`.
  **L2126 CN**: 执行以 `&expr{DEREF` 为核心的调用或声明。
- **L2127 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2127 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2128 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2128 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2129 EN**: Continues the surrounding expression or declaration: `d.rank == Rank::elementalOrBOZ) {`.
  **L2129 CN**: 继续构造周围的表达式或声明：`d.rank == Rank::elementalOrBOZ) {`。
- **L2130 EN**: Skips to the next loop iteration.
  **L2130 CN**: 跳到下一次循环迭代。
- **L2131 EN**: Transitions from the previous branch into the alternative path.
  **L2131 CN**: 从前一个分支过渡到备选路径。
- **L2132 EN**: Continues the surrounding expression or declaration: `const IntrinsicDummyArgument *nextParam{`.
  **L2132 CN**: 继续构造周围的表达式或声明：`const IntrinsicDummyArgument *nextParam{`。
- **L2133 EN**: Executes a standalone statement or declaration: `j + 1 < dummies ? &dummy[j + 1] : nullptr};`.
  **L2133 CN**: 执行一条独立语句或声明：`j + 1 < dummies ? &dummy[j + 1] : nullptr};`。
- **L2134 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2134 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2135 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2136 EN**: Continues logic associated with callable symbol `Typeless`.
  **L2136 CN**: 继续与可调用符号 `Typeless` 相关的逻辑。

### Lines 2137-2160

````cpp
                d.keyword, nextParam->keyword);
          } else {
            messages.Say(arg->sourceLocation(),
                "Typeless (BOZ) not allowed for '%s=' argument"_err_en_US,
                d.keyword);
          }
        }
      } else {
        // NULL(no MOLD=), procedure, or procedure pointer
        CHECK(IsProcedurePointerTarget(expr));
        if (d.typePattern.kindCode == KindCode::addressable ||
            d.rank == Rank::reduceOperation) {
          continue;
        } else if (d.typePattern.kindCode == KindCode::nullPointerType) {
          continue;
        } else if (IsBareNullPointer(&expr)) {
          // checked elsewhere
          continue;
        } else {
          CHECK(IsProcedure(expr) || IsProcedurePointer(expr));
          messages.Say(arg->sourceLocation(),
              "Actual argument for '%s=' may not be a procedure"_err_en_US,
              d.keyword);
        }
````
- **L2137 EN**: Executes a standalone statement or declaration: `d.keyword, nextParam->keyword);`.
  **L2137 CN**: 执行一条独立语句或声明：`d.keyword, nextParam->keyword);`。
- **L2138 EN**: Transitions from the previous branch into the alternative path.
  **L2138 CN**: 从前一个分支过渡到备选路径。
- **L2139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2139 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Typeless (BOZ) not allowed for '%s=' argument"_err_en_US,`.
  **L2140 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Typeless (BOZ) not allowed for '%s=' argument"_err_en_US,`。
- **L2141 EN**: Executes a standalone statement or declaration: `d.keyword);`.
  **L2141 CN**: 执行一条独立语句或声明：`d.keyword);`。
- **L2142 EN**: Closes the current lexical scope or compound statement.
  **L2142 CN**: 结束当前词法作用域或复合语句块。
- **L2143 EN**: Closes the current lexical scope or compound statement.
  **L2143 CN**: 结束当前词法作用域或复合语句块。
- **L2144 EN**: Transitions from the previous branch into the alternative path.
  **L2144 CN**: 从前一个分支过渡到备选路径。
- **L2145 EN**: Comment explains nearby logic, intent, or metadata: `NULL(no MOLD=), procedure, or procedure pointer`.
  **L2145 CN**: 注释说明附近代码的逻辑、意图或元数据：`NULL(no MOLD=), procedure, or procedure pointer`。
- **L2146 EN**: Executes a call or declaration centered on `CHECK`.
  **L2146 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2147 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2147 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2148 EN**: Continues the surrounding expression or declaration: `d.rank == Rank::reduceOperation) {`.
  **L2148 CN**: 继续构造周围的表达式或声明：`d.rank == Rank::reduceOperation) {`。
- **L2149 EN**: Skips to the next loop iteration.
  **L2149 CN**: 跳到下一次循环迭代。
- **L2150 EN**: Transitions from the previous branch into an `else if` condition.
  **L2150 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2151 EN**: Skips to the next loop iteration.
  **L2151 CN**: 跳到下一次循环迭代。
- **L2152 EN**: Transitions from the previous branch into an `else if` condition.
  **L2152 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2153 EN**: Comment explains nearby logic, intent, or metadata: `checked elsewhere`.
  **L2153 CN**: 注释说明附近代码的逻辑、意图或元数据：`checked elsewhere`。
- **L2154 EN**: Skips to the next loop iteration.
  **L2154 CN**: 跳到下一次循环迭代。
- **L2155 EN**: Transitions from the previous branch into the alternative path.
  **L2155 CN**: 从前一个分支过渡到备选路径。
- **L2156 EN**: Executes a call or declaration centered on `CHECK`.
  **L2156 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2157 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument for '%s=' may not be a procedure"_err_en_US,`.
  **L2158 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument for '%s=' may not be a procedure"_err_en_US,`。
- **L2159 EN**: Executes a standalone statement or declaration: `d.keyword);`.
  **L2159 CN**: 执行一条独立语句或声明：`d.keyword);`。
- **L2160 EN**: Closes the current lexical scope or compound statement.
  **L2160 CN**: 结束当前词法作用域或复合语句块。

### Lines 2161-2184

````cpp
      }
      return std::nullopt;
    } else if (!d.typePattern.categorySet.test(type->category())) {
      const char *expected{
          d.typePattern.kindCode == KindCode::extensibleOrUnlimitedType
              ? ", expected extensible or unlimited polymorphic type"
              : ""};
      messages.Say(arg->sourceLocation(),
          "Actual argument for '%s=' has bad type '%s'%s"_err_en_US, d.keyword,
          type->AsFortran(), expected);
      return std::nullopt; // argument has invalid type category
    }
    bool argOk{false};
    switch (d.typePattern.kindCode) {
    case KindCode::none:
    case KindCode::typeless:
      argOk = false;
      break;
    case KindCode::eventType:
      argOk = !type->IsUnlimitedPolymorphic() &&
          type->category() == TypeCategory::Derived &&
          semantics::IsEventType(&type->GetDerivedTypeSpec());
      break;
    case KindCode::ieeeFlagType:
````
- **L2161 EN**: Closes the current lexical scope or compound statement.
  **L2161 CN**: 结束当前词法作用域或复合语句块。
- **L2162 EN**: Returns from the current function with `std::nullopt`.
  **L2162 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2163 EN**: Transitions from the previous branch into an `else if` condition.
  **L2163 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2164 EN**: Continues the surrounding expression or declaration: `const char *expected{`.
  **L2164 CN**: 继续构造周围的表达式或声明：`const char *expected{`。
- **L2165 EN**: Continues the surrounding expression or declaration: `d.typePattern.kindCode == KindCode::extensibleOrUnlimitedType`.
  **L2165 CN**: 继续构造周围的表达式或声明：`d.typePattern.kindCode == KindCode::extensibleOrUnlimitedType`。
- **L2166 EN**: Continues the surrounding expression or declaration: `? ", expected extensible or unlimited polymorphic type"`.
  **L2166 CN**: 继续构造周围的表达式或声明：`? ", expected extensible or unlimited polymorphic type"`。
- **L2167 EN**: Executes a standalone statement or declaration: `: ""};`.
  **L2167 CN**: 执行一条独立语句或声明：`: ""};`。
- **L2168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2168 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument for '%s=' has bad type '%s'%s"_err_en_US, d.keyword,`.
  **L2169 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument for '%s=' has bad type '%s'%s"_err_en_US, d.keyword,`。
- **L2170 EN**: Executes a call or declaration centered on `type->AsFortran`.
  **L2170 CN**: 执行以 `type->AsFortran` 为核心的调用或声明。
- **L2171 EN**: Returns from the current function with `std::nullopt; // argument has invalid type category`.
  **L2171 CN**: 以 `std::nullopt; // argument has invalid type category` 从当前函数返回。
- **L2172 EN**: Closes the current lexical scope or compound statement.
  **L2172 CN**: 结束当前词法作用域或复合语句块。
- **L2173 EN**: Executes a standalone statement or declaration: `bool argOk{false};`.
  **L2173 CN**: 执行一条独立语句或声明：`bool argOk{false};`。
- **L2174 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2174 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2175 EN**: Introduces a switch dispatch label: `case KindCode::none:`.
  **L2175 CN**: 引入一个 switch 分发标签：`case KindCode::none:`。
- **L2176 EN**: Introduces a switch dispatch label: `case KindCode::typeless:`.
  **L2176 CN**: 引入一个 switch 分发标签：`case KindCode::typeless:`。
- **L2177 EN**: Executes a standalone statement or declaration: `argOk = false;`.
  **L2177 CN**: 执行一条独立语句或声明：`argOk = false;`。
- **L2178 EN**: Exits the nearest loop or switch statement.
  **L2178 CN**: 退出最近的循环或 switch 语句。
- **L2179 EN**: Introduces a switch dispatch label: `case KindCode::eventType:`.
  **L2179 CN**: 引入一个 switch 分发标签：`case KindCode::eventType:`。
- **L2180 EN**: Continues logic associated with callable symbol `IsUnlimitedPolymorphic`.
  **L2180 CN**: 继续与可调用符号 `IsUnlimitedPolymorphic` 相关的逻辑。
- **L2181 EN**: Continues logic associated with callable symbol `category`.
  **L2181 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L2182 EN**: Executes a call or declaration centered on `semantics::IsEventType`.
  **L2182 CN**: 执行以 `semantics::IsEventType` 为核心的调用或声明。
- **L2183 EN**: Exits the nearest loop or switch statement.
  **L2183 CN**: 退出最近的循环或 switch 语句。
- **L2184 EN**: Introduces a switch dispatch label: `case KindCode::ieeeFlagType:`.
  **L2184 CN**: 引入一个 switch 分发标签：`case KindCode::ieeeFlagType:`。

### Lines 2185-2208

````cpp
      argOk = !type->IsUnlimitedPolymorphic() &&
          type->category() == TypeCategory::Derived &&
          semantics::IsIeeeFlagType(&type->GetDerivedTypeSpec());
      break;
    case KindCode::ieeeRoundType:
      argOk = !type->IsUnlimitedPolymorphic() &&
          type->category() == TypeCategory::Derived &&
          semantics::IsIeeeRoundType(&type->GetDerivedTypeSpec());
      break;
    case KindCode::teamType:
      argOk = !type->IsUnlimitedPolymorphic() &&
          type->category() == TypeCategory::Derived &&
          semantics::IsTeamType(&type->GetDerivedTypeSpec());
      break;
    case KindCode::defaultIntegerKind:
      argOk = type->kind() == defaults.GetDefaultKind(TypeCategory::Integer);
      break;
    case KindCode::defaultRealKind:
      argOk = type->kind() == defaults.GetDefaultKind(TypeCategory::Real);
      break;
    case KindCode::doublePrecision:
      argOk = type->kind() == defaults.doublePrecisionKind();
      break;
    case KindCode::quadPrecision:
````
- **L2185 EN**: Continues logic associated with callable symbol `IsUnlimitedPolymorphic`.
  **L2185 CN**: 继续与可调用符号 `IsUnlimitedPolymorphic` 相关的逻辑。
- **L2186 EN**: Continues logic associated with callable symbol `category`.
  **L2186 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L2187 EN**: Executes a call or declaration centered on `semantics::IsIeeeFlagType`.
  **L2187 CN**: 执行以 `semantics::IsIeeeFlagType` 为核心的调用或声明。
- **L2188 EN**: Exits the nearest loop or switch statement.
  **L2188 CN**: 退出最近的循环或 switch 语句。
- **L2189 EN**: Introduces a switch dispatch label: `case KindCode::ieeeRoundType:`.
  **L2189 CN**: 引入一个 switch 分发标签：`case KindCode::ieeeRoundType:`。
- **L2190 EN**: Continues logic associated with callable symbol `IsUnlimitedPolymorphic`.
  **L2190 CN**: 继续与可调用符号 `IsUnlimitedPolymorphic` 相关的逻辑。
- **L2191 EN**: Continues logic associated with callable symbol `category`.
  **L2191 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L2192 EN**: Executes a call or declaration centered on `semantics::IsIeeeRoundType`.
  **L2192 CN**: 执行以 `semantics::IsIeeeRoundType` 为核心的调用或声明。
- **L2193 EN**: Exits the nearest loop or switch statement.
  **L2193 CN**: 退出最近的循环或 switch 语句。
- **L2194 EN**: Introduces a switch dispatch label: `case KindCode::teamType:`.
  **L2194 CN**: 引入一个 switch 分发标签：`case KindCode::teamType:`。
- **L2195 EN**: Continues logic associated with callable symbol `IsUnlimitedPolymorphic`.
  **L2195 CN**: 继续与可调用符号 `IsUnlimitedPolymorphic` 相关的逻辑。
- **L2196 EN**: Continues logic associated with callable symbol `category`.
  **L2196 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L2197 EN**: Executes a call or declaration centered on `semantics::IsTeamType`.
  **L2197 CN**: 执行以 `semantics::IsTeamType` 为核心的调用或声明。
- **L2198 EN**: Exits the nearest loop or switch statement.
  **L2198 CN**: 退出最近的循环或 switch 语句。
- **L2199 EN**: Introduces a switch dispatch label: `case KindCode::defaultIntegerKind:`.
  **L2199 CN**: 引入一个 switch 分发标签：`case KindCode::defaultIntegerKind:`。
- **L2200 EN**: Executes a call or declaration centered on `type->kind`.
  **L2200 CN**: 执行以 `type->kind` 为核心的调用或声明。
- **L2201 EN**: Exits the nearest loop or switch statement.
  **L2201 CN**: 退出最近的循环或 switch 语句。
- **L2202 EN**: Introduces a switch dispatch label: `case KindCode::defaultRealKind:`.
  **L2202 CN**: 引入一个 switch 分发标签：`case KindCode::defaultRealKind:`。
- **L2203 EN**: Executes a call or declaration centered on `type->kind`.
  **L2203 CN**: 执行以 `type->kind` 为核心的调用或声明。
- **L2204 EN**: Exits the nearest loop or switch statement.
  **L2204 CN**: 退出最近的循环或 switch 语句。
- **L2205 EN**: Introduces a switch dispatch label: `case KindCode::doublePrecision:`.
  **L2205 CN**: 引入一个 switch 分发标签：`case KindCode::doublePrecision:`。
- **L2206 EN**: Executes a call or declaration centered on `type->kind`.
  **L2206 CN**: 执行以 `type->kind` 为核心的调用或声明。
- **L2207 EN**: Exits the nearest loop or switch statement.
  **L2207 CN**: 退出最近的循环或 switch 语句。
- **L2208 EN**: Introduces a switch dispatch label: `case KindCode::quadPrecision:`.
  **L2208 CN**: 引入一个 switch 分发标签：`case KindCode::quadPrecision:`。

### Lines 2209-2232

````cpp
      argOk = type->kind() == defaults.quadPrecisionKind();
      break;
    case KindCode::defaultCharKind:
      argOk = type->kind() == defaults.GetDefaultKind(TypeCategory::Character);
      break;
    case KindCode::defaultLogicalKind:
      argOk = type->kind() == defaults.GetDefaultKind(TypeCategory::Logical);
      break;
    case KindCode::any:
      argOk = true;
      break;
    case KindCode::kindArg:
      CHECK(type->category() == TypeCategory::Integer);
      CHECK(!kindArg);
      kindArg = arg;
      argOk = true;
      break;
    case KindCode::dimArg:
      CHECK(type->category() == TypeCategory::Integer);
      dimArg = j;
      argOk = true;
      break;
    case KindCode::same: {
      if (!sameArg) {
````
- **L2209 EN**: Executes a call or declaration centered on `type->kind`.
  **L2209 CN**: 执行以 `type->kind` 为核心的调用或声明。
- **L2210 EN**: Exits the nearest loop or switch statement.
  **L2210 CN**: 退出最近的循环或 switch 语句。
- **L2211 EN**: Introduces a switch dispatch label: `case KindCode::defaultCharKind:`.
  **L2211 CN**: 引入一个 switch 分发标签：`case KindCode::defaultCharKind:`。
- **L2212 EN**: Executes a call or declaration centered on `type->kind`.
  **L2212 CN**: 执行以 `type->kind` 为核心的调用或声明。
- **L2213 EN**: Exits the nearest loop or switch statement.
  **L2213 CN**: 退出最近的循环或 switch 语句。
- **L2214 EN**: Introduces a switch dispatch label: `case KindCode::defaultLogicalKind:`.
  **L2214 CN**: 引入一个 switch 分发标签：`case KindCode::defaultLogicalKind:`。
- **L2215 EN**: Executes a call or declaration centered on `type->kind`.
  **L2215 CN**: 执行以 `type->kind` 为核心的调用或声明。
- **L2216 EN**: Exits the nearest loop or switch statement.
  **L2216 CN**: 退出最近的循环或 switch 语句。
- **L2217 EN**: Introduces a switch dispatch label: `case KindCode::any:`.
  **L2217 CN**: 引入一个 switch 分发标签：`case KindCode::any:`。
- **L2218 EN**: Executes a standalone statement or declaration: `argOk = true;`.
  **L2218 CN**: 执行一条独立语句或声明：`argOk = true;`。
- **L2219 EN**: Exits the nearest loop or switch statement.
  **L2219 CN**: 退出最近的循环或 switch 语句。
- **L2220 EN**: Introduces a switch dispatch label: `case KindCode::kindArg:`.
  **L2220 CN**: 引入一个 switch 分发标签：`case KindCode::kindArg:`。
- **L2221 EN**: Executes a call or declaration centered on `CHECK`.
  **L2221 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2222 EN**: Executes a call or declaration centered on `CHECK`.
  **L2222 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2223 EN**: Executes a standalone statement or declaration: `kindArg = arg;`.
  **L2223 CN**: 执行一条独立语句或声明：`kindArg = arg;`。
- **L2224 EN**: Executes a standalone statement or declaration: `argOk = true;`.
  **L2224 CN**: 执行一条独立语句或声明：`argOk = true;`。
- **L2225 EN**: Exits the nearest loop or switch statement.
  **L2225 CN**: 退出最近的循环或 switch 语句。
- **L2226 EN**: Introduces a switch dispatch label: `case KindCode::dimArg:`.
  **L2226 CN**: 引入一个 switch 分发标签：`case KindCode::dimArg:`。
- **L2227 EN**: Executes a call or declaration centered on `CHECK`.
  **L2227 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2228 EN**: Executes a standalone statement or declaration: `dimArg = j;`.
  **L2228 CN**: 执行一条独立语句或声明：`dimArg = j;`。
- **L2229 EN**: Executes a standalone statement or declaration: `argOk = true;`.
  **L2229 CN**: 执行一条独立语句或声明：`argOk = true;`。
- **L2230 EN**: Exits the nearest loop or switch statement.
  **L2230 CN**: 退出最近的循环或 switch 语句。
- **L2231 EN**: Introduces a switch dispatch label: `case KindCode::same: {`.
  **L2231 CN**: 引入一个 switch 分发标签：`case KindCode::same: {`。
- **L2232 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2232 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2233-2256

````cpp
        sameArg = arg;
      }
      auto sameType{sameArg->GetType().value()};
      if (name == "move_alloc"s) {
        // second argument can be more general
        argOk = type->IsTkLenCompatibleWith(sameType);
      } else if (name == "merge"s) {
        argOk = type->IsTkLenCompatibleWith(sameType) &&
            sameType.IsTkLenCompatibleWith(*type);
      } else {
        argOk = sameType.IsTkLenCompatibleWith(*type);
      }
    } break;
    case KindCode::sameKind:
      if (!sameArg) {
        sameArg = arg;
      }
      argOk = type->IsTkCompatibleWith(sameArg->GetType().value());
      break;
    case KindCode::operand:
      if (!operandArg) {
        operandArg = arg;
      } else if (auto prev{operandArg->GetType()}) {
        if (type->category() == prev->category()) {
````
- **L2233 EN**: Executes a standalone statement or declaration: `sameArg = arg;`.
  **L2233 CN**: 执行一条独立语句或声明：`sameArg = arg;`。
- **L2234 EN**: Closes the current lexical scope or compound statement.
  **L2234 CN**: 结束当前词法作用域或复合语句块。
- **L2235 EN**: Executes a call or declaration centered on `sameType{sameArg->GetType`.
  **L2235 CN**: 执行以 `sameType{sameArg->GetType` 为核心的调用或声明。
- **L2236 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2236 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2237 EN**: Comment explains nearby logic, intent, or metadata: `second argument can be more general`.
  **L2237 CN**: 注释说明附近代码的逻辑、意图或元数据：`second argument can be more general`。
- **L2238 EN**: Executes a call or declaration centered on `type->IsTkLenCompatibleWith`.
  **L2238 CN**: 执行以 `type->IsTkLenCompatibleWith` 为核心的调用或声明。
- **L2239 EN**: Transitions from the previous branch into an `else if` condition.
  **L2239 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2240 EN**: Continues logic associated with callable symbol `IsTkLenCompatibleWith`.
  **L2240 CN**: 继续与可调用符号 `IsTkLenCompatibleWith` 相关的逻辑。
- **L2241 EN**: Executes a call or declaration centered on `sameType.IsTkLenCompatibleWith`.
  **L2241 CN**: 执行以 `sameType.IsTkLenCompatibleWith` 为核心的调用或声明。
- **L2242 EN**: Transitions from the previous branch into the alternative path.
  **L2242 CN**: 从前一个分支过渡到备选路径。
- **L2243 EN**: Executes a call or declaration centered on `sameType.IsTkLenCompatibleWith`.
  **L2243 CN**: 执行以 `sameType.IsTkLenCompatibleWith` 为核心的调用或声明。
- **L2244 EN**: Closes the current lexical scope or compound statement.
  **L2244 CN**: 结束当前词法作用域或复合语句块。
- **L2245 EN**: Executes a standalone statement or declaration: `} break;`.
  **L2245 CN**: 执行一条独立语句或声明：`} break;`。
- **L2246 EN**: Introduces a switch dispatch label: `case KindCode::sameKind:`.
  **L2246 CN**: 引入一个 switch 分发标签：`case KindCode::sameKind:`。
- **L2247 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2247 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2248 EN**: Executes a standalone statement or declaration: `sameArg = arg;`.
  **L2248 CN**: 执行一条独立语句或声明：`sameArg = arg;`。
- **L2249 EN**: Closes the current lexical scope or compound statement.
  **L2249 CN**: 结束当前词法作用域或复合语句块。
- **L2250 EN**: Executes a call or declaration centered on `type->IsTkCompatibleWith`.
  **L2250 CN**: 执行以 `type->IsTkCompatibleWith` 为核心的调用或声明。
- **L2251 EN**: Exits the nearest loop or switch statement.
  **L2251 CN**: 退出最近的循环或 switch 语句。
- **L2252 EN**: Introduces a switch dispatch label: `case KindCode::operand:`.
  **L2252 CN**: 引入一个 switch 分发标签：`case KindCode::operand:`。
- **L2253 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2253 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2254 EN**: Executes a standalone statement or declaration: `operandArg = arg;`.
  **L2254 CN**: 执行一条独立语句或声明：`operandArg = arg;`。
- **L2255 EN**: Transitions from the previous branch into an `else if` condition.
  **L2255 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2256 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2256 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2257-2280

````cpp
          if (type->kind() > prev->kind()) {
            operandArg = arg;
          }
        } else if (prev->category() == TypeCategory::Integer) {
          operandArg = arg;
        }
      }
      argOk = true;
      break;
    case KindCode::effectiveKind:
      common::die("INTERNAL: KindCode::effectiveKind appears on argument '%s' "
                  "for intrinsic '%s'",
          d.keyword, name);
      break;
    case KindCode::addressable:
    case KindCode::nullPointerType:
      argOk = true;
      break;
    case KindCode::exactKind:
      argOk = type->kind() == d.typePattern.kindValue;
      break;
    case KindCode::greaterOrEqualToKind:
      argOk = type->kind() >= d.typePattern.kindValue;
      break;
````
- **L2257 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2257 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2258 EN**: Executes a standalone statement or declaration: `operandArg = arg;`.
  **L2258 CN**: 执行一条独立语句或声明：`operandArg = arg;`。
- **L2259 EN**: Closes the current lexical scope or compound statement.
  **L2259 CN**: 结束当前词法作用域或复合语句块。
- **L2260 EN**: Transitions from the previous branch into an `else if` condition.
  **L2260 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2261 EN**: Executes a standalone statement or declaration: `operandArg = arg;`.
  **L2261 CN**: 执行一条独立语句或声明：`operandArg = arg;`。
- **L2262 EN**: Closes the current lexical scope or compound statement.
  **L2262 CN**: 结束当前词法作用域或复合语句块。
- **L2263 EN**: Closes the current lexical scope or compound statement.
  **L2263 CN**: 结束当前词法作用域或复合语句块。
- **L2264 EN**: Executes a standalone statement or declaration: `argOk = true;`.
  **L2264 CN**: 执行一条独立语句或声明：`argOk = true;`。
- **L2265 EN**: Exits the nearest loop or switch statement.
  **L2265 CN**: 退出最近的循环或 switch 语句。
- **L2266 EN**: Introduces a switch dispatch label: `case KindCode::effectiveKind:`.
  **L2266 CN**: 引入一个 switch 分发标签：`case KindCode::effectiveKind:`。
- **L2267 EN**: Continues logic associated with callable symbol `die`.
  **L2267 CN**: 继续与可调用符号 `die` 相关的逻辑。
- **L2268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"for intrinsic '%s'",`.
  **L2268 CN**: 继续一个多行参数列表、初始化器或聚合项：`"for intrinsic '%s'",`。
- **L2269 EN**: Executes a standalone statement or declaration: `d.keyword, name);`.
  **L2269 CN**: 执行一条独立语句或声明：`d.keyword, name);`。
- **L2270 EN**: Exits the nearest loop or switch statement.
  **L2270 CN**: 退出最近的循环或 switch 语句。
- **L2271 EN**: Introduces a switch dispatch label: `case KindCode::addressable:`.
  **L2271 CN**: 引入一个 switch 分发标签：`case KindCode::addressable:`。
- **L2272 EN**: Introduces a switch dispatch label: `case KindCode::nullPointerType:`.
  **L2272 CN**: 引入一个 switch 分发标签：`case KindCode::nullPointerType:`。
- **L2273 EN**: Executes a standalone statement or declaration: `argOk = true;`.
  **L2273 CN**: 执行一条独立语句或声明：`argOk = true;`。
- **L2274 EN**: Exits the nearest loop or switch statement.
  **L2274 CN**: 退出最近的循环或 switch 语句。
- **L2275 EN**: Introduces a switch dispatch label: `case KindCode::exactKind:`.
  **L2275 CN**: 引入一个 switch 分发标签：`case KindCode::exactKind:`。
- **L2276 EN**: Executes a call or declaration centered on `type->kind`.
  **L2276 CN**: 执行以 `type->kind` 为核心的调用或声明。
- **L2277 EN**: Exits the nearest loop or switch statement.
  **L2277 CN**: 退出最近的循环或 switch 语句。
- **L2278 EN**: Introduces a switch dispatch label: `case KindCode::greaterOrEqualToKind:`.
  **L2278 CN**: 引入一个 switch 分发标签：`case KindCode::greaterOrEqualToKind:`。
- **L2279 EN**: Executes a call or declaration centered on `type->kind`.
  **L2279 CN**: 执行以 `type->kind` 为核心的调用或声明。
- **L2280 EN**: Exits the nearest loop or switch statement.
  **L2280 CN**: 退出最近的循环或 switch 语句。

### Lines 2281-2304

````cpp
    case KindCode::sameAtom:
      if (!sameArg) {
        sameArg = arg;
        argOk = CheckAtomicKind(DEREF(arg), builtinsScope, messages, d.keyword);
      } else {
        argOk = type->IsTkCompatibleWith(sameArg->GetType().value());
        if (!argOk) {
          messages.Say(arg->sourceLocation(),
              "Actual argument for '%s=' must have same type and kind as 'atom=', but is '%s'"_err_en_US,
              d.keyword, type->AsFortran());
        }
      }
      if (!argOk) {
        return std::nullopt;
      }
      break;
    case KindCode::atomicIntKind:
      argOk = CheckAtomicKind(DEREF(arg), builtinsScope, messages, d.keyword);
      if (!argOk) {
        return std::nullopt;
      }
      break;
    case KindCode::atomicIntOrLogicalKind:
      argOk = CheckAtomicKind(DEREF(arg), builtinsScope, messages, d.keyword);
````
- **L2281 EN**: Introduces a switch dispatch label: `case KindCode::sameAtom:`.
  **L2281 CN**: 引入一个 switch 分发标签：`case KindCode::sameAtom:`。
- **L2282 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2282 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2283 EN**: Executes a standalone statement or declaration: `sameArg = arg;`.
  **L2283 CN**: 执行一条独立语句或声明：`sameArg = arg;`。
- **L2284 EN**: Executes a call or declaration centered on `CheckAtomicKind`.
  **L2284 CN**: 执行以 `CheckAtomicKind` 为核心的调用或声明。
- **L2285 EN**: Transitions from the previous branch into the alternative path.
  **L2285 CN**: 从前一个分支过渡到备选路径。
- **L2286 EN**: Executes a call or declaration centered on `type->IsTkCompatibleWith`.
  **L2286 CN**: 执行以 `type->IsTkCompatibleWith` 为核心的调用或声明。
- **L2287 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2287 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2288 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument for '%s=' must have same type and kind as 'atom=', but is '%s'"_err_en_US,`.
  **L2289 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument for '%s=' must have same type and kind as 'atom=', but is '%s'"_err_en_US,`。
- **L2290 EN**: Executes a call or declaration centered on `type->AsFortran`.
  **L2290 CN**: 执行以 `type->AsFortran` 为核心的调用或声明。
- **L2291 EN**: Closes the current lexical scope or compound statement.
  **L2291 CN**: 结束当前词法作用域或复合语句块。
- **L2292 EN**: Closes the current lexical scope or compound statement.
  **L2292 CN**: 结束当前词法作用域或复合语句块。
- **L2293 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2293 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2294 EN**: Returns from the current function with `std::nullopt`.
  **L2294 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2295 EN**: Closes the current lexical scope or compound statement.
  **L2295 CN**: 结束当前词法作用域或复合语句块。
- **L2296 EN**: Exits the nearest loop or switch statement.
  **L2296 CN**: 退出最近的循环或 switch 语句。
- **L2297 EN**: Introduces a switch dispatch label: `case KindCode::atomicIntKind:`.
  **L2297 CN**: 引入一个 switch 分发标签：`case KindCode::atomicIntKind:`。
- **L2298 EN**: Executes a call or declaration centered on `CheckAtomicKind`.
  **L2298 CN**: 执行以 `CheckAtomicKind` 为核心的调用或声明。
- **L2299 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2299 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2300 EN**: Returns from the current function with `std::nullopt`.
  **L2300 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2301 EN**: Closes the current lexical scope or compound statement.
  **L2301 CN**: 结束当前词法作用域或复合语句块。
- **L2302 EN**: Exits the nearest loop or switch statement.
  **L2302 CN**: 退出最近的循环或 switch 语句。
- **L2303 EN**: Introduces a switch dispatch label: `case KindCode::atomicIntOrLogicalKind:`.
  **L2303 CN**: 引入一个 switch 分发标签：`case KindCode::atomicIntOrLogicalKind:`。
- **L2304 EN**: Executes a call or declaration centered on `CheckAtomicKind`.
  **L2304 CN**: 执行以 `CheckAtomicKind` 为核心的调用或声明。

### Lines 2305-2328

````cpp
      if (!argOk) {
        return std::nullopt;
      }
      break;
    case KindCode::extensibleOrUnlimitedType:
      argOk = type->IsUnlimitedPolymorphic() ||
          (type->category() == TypeCategory::Derived &&
              IsExtensibleType(GetDerivedTypeSpec(type)));
      if (!argOk) {
        messages.Say(arg->sourceLocation(),
            "Actual argument for '%s=' has type '%s', but was expected to be an extensible or unlimited polymorphic type"_err_en_US,
            d.keyword, type->AsFortran());
        return std::nullopt;
      }
      break;
    default:
      CRASH_NO_CASE;
    }
    if (!argOk) {
      messages.Say(arg->sourceLocation(),
          "Actual argument for '%s=' has bad type or kind '%s'"_err_en_US,
          d.keyword, type->AsFortran());
      return std::nullopt;
    }
````
- **L2305 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2305 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2306 EN**: Returns from the current function with `std::nullopt`.
  **L2306 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2307 EN**: Closes the current lexical scope or compound statement.
  **L2307 CN**: 结束当前词法作用域或复合语句块。
- **L2308 EN**: Exits the nearest loop or switch statement.
  **L2308 CN**: 退出最近的循环或 switch 语句。
- **L2309 EN**: Introduces a switch dispatch label: `case KindCode::extensibleOrUnlimitedType:`.
  **L2309 CN**: 引入一个 switch 分发标签：`case KindCode::extensibleOrUnlimitedType:`。
- **L2310 EN**: Continues logic associated with callable symbol `IsUnlimitedPolymorphic`.
  **L2310 CN**: 继续与可调用符号 `IsUnlimitedPolymorphic` 相关的逻辑。
- **L2311 EN**: Continues logic associated with callable symbol `category`.
  **L2311 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L2312 EN**: Executes a call or declaration centered on `IsExtensibleType`.
  **L2312 CN**: 执行以 `IsExtensibleType` 为核心的调用或声明。
- **L2313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2314 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument for '%s=' has type '%s', but was expected to be an extensible or unlimited polymorphic type"_err_en_US,`.
  **L2315 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument for '%s=' has type '%s', but was expected to be an extensible or unlimited polymorphic type"_err_en_US,`。
- **L2316 EN**: Executes a call or declaration centered on `type->AsFortran`.
  **L2316 CN**: 执行以 `type->AsFortran` 为核心的调用或声明。
- **L2317 EN**: Returns from the current function with `std::nullopt`.
  **L2317 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2318 EN**: Closes the current lexical scope or compound statement.
  **L2318 CN**: 结束当前词法作用域或复合语句块。
- **L2319 EN**: Exits the nearest loop or switch statement.
  **L2319 CN**: 退出最近的循环或 switch 语句。
- **L2320 EN**: Introduces a switch dispatch label: `default:`.
  **L2320 CN**: 引入一个 switch 分发标签：`default:`。
- **L2321 EN**: Executes a standalone statement or declaration: `CRASH_NO_CASE;`.
  **L2321 CN**: 执行一条独立语句或声明：`CRASH_NO_CASE;`。
- **L2322 EN**: Closes the current lexical scope or compound statement.
  **L2322 CN**: 结束当前词法作用域或复合语句块。
- **L2323 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2323 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2324 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument for '%s=' has bad type or kind '%s'"_err_en_US,`.
  **L2325 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument for '%s=' has bad type or kind '%s'"_err_en_US,`。
- **L2326 EN**: Executes a call or declaration centered on `type->AsFortran`.
  **L2326 CN**: 执行以 `type->AsFortran` 为核心的调用或声明。
- **L2327 EN**: Returns from the current function with `std::nullopt`.
  **L2327 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2328 EN**: Closes the current lexical scope or compound statement.
  **L2328 CN**: 结束当前词法作用域或复合语句块。

### Lines 2329-2352

````cpp
  }

  // Check the ranks of the arguments against the intrinsic's interface.
  const ActualArgument *arrayArg{nullptr};
  const char *arrayArgName{nullptr};
  const ActualArgument *knownArg{nullptr};
  std::optional<std::int64_t> shapeArgSize;
  int elementalRank{0};
  for (std::size_t j{0}; j < dummies; ++j) {
    const IntrinsicDummyArgument &d{dummy[std::min(j, dummyArgPatterns - 1)]};
    if (const ActualArgument *arg{actualForDummy[j]}) {
      bool isAssumedRank{semantics::IsAssumedRank(*arg)};
      if (isAssumedRank && d.rank != Rank::anyOrAssumedRank &&
          d.rank != Rank::arrayOrAssumedRank) {
        messages.Say(arg->sourceLocation(),
            "Assumed-rank array cannot be forwarded to '%s=' argument"_err_en_US,
            d.keyword);
        return std::nullopt;
      }
      int rank{arg->Rank()};
      bool argOk{false};
      switch (d.rank) {
      case Rank::elemental:
      case Rank::elementalOrBOZ:
````
- **L2329 EN**: Closes the current lexical scope or compound statement.
  **L2329 CN**: 结束当前词法作用域或复合语句块。
- **L2330 EN**: Blank line separating nearby declarations or logic blocks.
  **L2330 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2331 EN**: Comment explains nearby logic, intent, or metadata: `Check the ranks of the arguments against the intrinsic's interface.`.
  **L2331 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check the ranks of the arguments against the intrinsic's interface.`。
- **L2332 EN**: Executes a standalone statement or declaration: `const ActualArgument *arrayArg{nullptr};`.
  **L2332 CN**: 执行一条独立语句或声明：`const ActualArgument *arrayArg{nullptr};`。
- **L2333 EN**: Executes a standalone statement or declaration: `const char *arrayArgName{nullptr};`.
  **L2333 CN**: 执行一条独立语句或声明：`const char *arrayArgName{nullptr};`。
- **L2334 EN**: Executes a standalone statement or declaration: `const ActualArgument *knownArg{nullptr};`.
  **L2334 CN**: 执行一条独立语句或声明：`const ActualArgument *knownArg{nullptr};`。
- **L2335 EN**: Executes a standalone statement or declaration: `std::optional<std::int64_t> shapeArgSize;`.
  **L2335 CN**: 执行一条独立语句或声明：`std::optional<std::int64_t> shapeArgSize;`。
- **L2336 EN**: Executes a standalone statement or declaration: `int elementalRank{0};`.
  **L2336 CN**: 执行一条独立语句或声明：`int elementalRank{0};`。
- **L2337 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2337 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2338 EN**: Executes a call or declaration centered on `&d{dummy[std::min`.
  **L2338 CN**: 执行以 `&d{dummy[std::min` 为核心的调用或声明。
- **L2339 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2339 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2340 EN**: Executes a call or declaration centered on `isAssumedRank{semantics::IsAssumedRank`.
  **L2340 CN**: 执行以 `isAssumedRank{semantics::IsAssumedRank` 为核心的调用或声明。
- **L2341 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2341 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2342 EN**: Continues the surrounding expression or declaration: `d.rank != Rank::arrayOrAssumedRank) {`.
  **L2342 CN**: 继续构造周围的表达式或声明：`d.rank != Rank::arrayOrAssumedRank) {`。
- **L2343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2343 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-rank array cannot be forwarded to '%s=' argument"_err_en_US,`.
  **L2344 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-rank array cannot be forwarded to '%s=' argument"_err_en_US,`。
- **L2345 EN**: Executes a standalone statement or declaration: `d.keyword);`.
  **L2345 CN**: 执行一条独立语句或声明：`d.keyword);`。
- **L2346 EN**: Returns from the current function with `std::nullopt`.
  **L2346 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2347 EN**: Closes the current lexical scope or compound statement.
  **L2347 CN**: 结束当前词法作用域或复合语句块。
- **L2348 EN**: Executes a call or declaration centered on `rank{arg->Rank`.
  **L2348 CN**: 执行以 `rank{arg->Rank` 为核心的调用或声明。
- **L2349 EN**: Executes a standalone statement or declaration: `bool argOk{false};`.
  **L2349 CN**: 执行一条独立语句或声明：`bool argOk{false};`。
- **L2350 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2350 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2351 EN**: Introduces a switch dispatch label: `case Rank::elemental:`.
  **L2351 CN**: 引入一个 switch 分发标签：`case Rank::elemental:`。
- **L2352 EN**: Introduces a switch dispatch label: `case Rank::elementalOrBOZ:`.
  **L2352 CN**: 引入一个 switch 分发标签：`case Rank::elementalOrBOZ:`。

### Lines 2353-2376

````cpp
        if (elementalRank == 0) {
          elementalRank = rank;
        }
        argOk = rank == 0 || rank == elementalRank;
        break;
      case Rank::scalar:
        argOk = rank == 0;
        break;
      case Rank::vector:
        argOk = rank == 1;
        break;
      case Rank::shape:
        CHECK(!shapeArgSize);
        if (rank != 1) {
          messages.Say(arg->sourceLocation(),
              "'shape=' argument must be an array of rank 1"_err_en_US);
          return std::nullopt;
        } else {
          if (auto shape{GetShape(context, *arg)}) {
            if (auto constShape{AsConstantShape(context, *shape)}) {
              shapeArgSize = constShape->At(ConstantSubscripts{1}).ToInt64();
              CHECK(shapeArgSize.value() >= 0);
              argOk = *shapeArgSize <= common::maxRank;
            }
````
- **L2353 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2353 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2354 EN**: Executes a standalone statement or declaration: `elementalRank = rank;`.
  **L2354 CN**: 执行一条独立语句或声明：`elementalRank = rank;`。
- **L2355 EN**: Closes the current lexical scope or compound statement.
  **L2355 CN**: 结束当前词法作用域或复合语句块。
- **L2356 EN**: Executes a standalone statement or declaration: `argOk = rank == 0 || rank == elementalRank;`.
  **L2356 CN**: 执行一条独立语句或声明：`argOk = rank == 0 || rank == elementalRank;`。
- **L2357 EN**: Exits the nearest loop or switch statement.
  **L2357 CN**: 退出最近的循环或 switch 语句。
- **L2358 EN**: Introduces a switch dispatch label: `case Rank::scalar:`.
  **L2358 CN**: 引入一个 switch 分发标签：`case Rank::scalar:`。
- **L2359 EN**: Executes a standalone statement or declaration: `argOk = rank == 0;`.
  **L2359 CN**: 执行一条独立语句或声明：`argOk = rank == 0;`。
- **L2360 EN**: Exits the nearest loop or switch statement.
  **L2360 CN**: 退出最近的循环或 switch 语句。
- **L2361 EN**: Introduces a switch dispatch label: `case Rank::vector:`.
  **L2361 CN**: 引入一个 switch 分发标签：`case Rank::vector:`。
- **L2362 EN**: Executes a standalone statement or declaration: `argOk = rank == 1;`.
  **L2362 CN**: 执行一条独立语句或声明：`argOk = rank == 1;`。
- **L2363 EN**: Exits the nearest loop or switch statement.
  **L2363 CN**: 退出最近的循环或 switch 语句。
- **L2364 EN**: Introduces a switch dispatch label: `case Rank::shape:`.
  **L2364 CN**: 引入一个 switch 分发标签：`case Rank::shape:`。
- **L2365 EN**: Executes a call or declaration centered on `CHECK`.
  **L2365 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2366 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2366 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2367 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2368 EN**: Executes a standalone statement or declaration: `"'shape=' argument must be an array of rank 1"_err_en_US);`.
  **L2368 CN**: 执行一条独立语句或声明：`"'shape=' argument must be an array of rank 1"_err_en_US);`。
- **L2369 EN**: Returns from the current function with `std::nullopt`.
  **L2369 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2370 EN**: Transitions from the previous branch into the alternative path.
  **L2370 CN**: 从前一个分支过渡到备选路径。
- **L2371 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2371 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2372 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2372 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2373 EN**: Executes a call or declaration centered on `constShape->At`.
  **L2373 CN**: 执行以 `constShape->At` 为核心的调用或声明。
- **L2374 EN**: Executes a call or declaration centered on `CHECK`.
  **L2374 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2375 EN**: Executes a standalone statement or declaration: `argOk = *shapeArgSize <= common::maxRank;`.
  **L2375 CN**: 执行一条独立语句或声明：`argOk = *shapeArgSize <= common::maxRank;`。
- **L2376 EN**: Closes the current lexical scope or compound statement.
  **L2376 CN**: 结束当前词法作用域或复合语句块。

### Lines 2377-2400

````cpp
          }
        }
        if (!argOk) {
          if (shapeArgSize.value_or(0) > common::maxRank) {
            messages.Say(arg->sourceLocation(),
                "'shape=' argument must be a vector of at most %d elements (has %jd)"_err_en_US,
                common::maxRank, std::intmax_t{*shapeArgSize});
          } else {
            messages.Say(arg->sourceLocation(),
                "'shape=' argument must be a vector of known size"_err_en_US);
          }
          return std::nullopt;
        }
        break;
      case Rank::matrix:
        argOk = rank == 2;
        break;
      case Rank::array:
        argOk = rank > 0;
        if (!arrayArg) {
          arrayArg = arg;
          arrayArgName = d.keyword;
        }
        break;
````
- **L2377 EN**: Closes the current lexical scope or compound statement.
  **L2377 CN**: 结束当前词法作用域或复合语句块。
- **L2378 EN**: Closes the current lexical scope or compound statement.
  **L2378 CN**: 结束当前词法作用域或复合语句块。
- **L2379 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2379 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2380 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2380 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2381 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'shape=' argument must be a vector of at most %d elements (has %jd)"_err_en_US,`.
  **L2382 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'shape=' argument must be a vector of at most %d elements (has %jd)"_err_en_US,`。
- **L2383 EN**: Executes a standalone statement or declaration: `common::maxRank, std::intmax_t{*shapeArgSize});`.
  **L2383 CN**: 执行一条独立语句或声明：`common::maxRank, std::intmax_t{*shapeArgSize});`。
- **L2384 EN**: Transitions from the previous branch into the alternative path.
  **L2384 CN**: 从前一个分支过渡到备选路径。
- **L2385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2385 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2386 EN**: Executes a standalone statement or declaration: `"'shape=' argument must be a vector of known size"_err_en_US);`.
  **L2386 CN**: 执行一条独立语句或声明：`"'shape=' argument must be a vector of known size"_err_en_US);`。
- **L2387 EN**: Closes the current lexical scope or compound statement.
  **L2387 CN**: 结束当前词法作用域或复合语句块。
- **L2388 EN**: Returns from the current function with `std::nullopt`.
  **L2388 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2389 EN**: Closes the current lexical scope or compound statement.
  **L2389 CN**: 结束当前词法作用域或复合语句块。
- **L2390 EN**: Exits the nearest loop or switch statement.
  **L2390 CN**: 退出最近的循环或 switch 语句。
- **L2391 EN**: Introduces a switch dispatch label: `case Rank::matrix:`.
  **L2391 CN**: 引入一个 switch 分发标签：`case Rank::matrix:`。
- **L2392 EN**: Executes a standalone statement or declaration: `argOk = rank == 2;`.
  **L2392 CN**: 执行一条独立语句或声明：`argOk = rank == 2;`。
- **L2393 EN**: Exits the nearest loop or switch statement.
  **L2393 CN**: 退出最近的循环或 switch 语句。
- **L2394 EN**: Introduces a switch dispatch label: `case Rank::array:`.
  **L2394 CN**: 引入一个 switch 分发标签：`case Rank::array:`。
- **L2395 EN**: Executes a standalone statement or declaration: `argOk = rank > 0;`.
  **L2395 CN**: 执行一条独立语句或声明：`argOk = rank > 0;`。
- **L2396 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2396 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2397 EN**: Executes a standalone statement or declaration: `arrayArg = arg;`.
  **L2397 CN**: 执行一条独立语句或声明：`arrayArg = arg;`。
- **L2398 EN**: Executes a standalone statement or declaration: `arrayArgName = d.keyword;`.
  **L2398 CN**: 执行一条独立语句或声明：`arrayArgName = d.keyword;`。
- **L2399 EN**: Closes the current lexical scope or compound statement.
  **L2399 CN**: 结束当前词法作用域或复合语句块。
- **L2400 EN**: Exits the nearest loop or switch statement.
  **L2400 CN**: 退出最近的循环或 switch 语句。

### Lines 2401-2424

````cpp
      case Rank::coarray:
        argOk = IsCoarray(*arg);
        if (!argOk) {
          messages.Say(arg->sourceLocation(),
              "'coarray=' argument must have corank > 0 for intrinsic '%s'"_err_en_US,
              name);
          return std::nullopt;
        }
        break;
      case Rank::atom:
        argOk = rank == 0 && (IsCoarray(*arg) || ExtractCoarrayRef(*arg));
        if (!argOk) {
          messages.Say(arg->sourceLocation(),
              "'%s=' argument must be a scalar coarray or coindexed object for intrinsic '%s'"_err_en_US,
              d.keyword, name);
          return std::nullopt;
        }
        break;
      case Rank::known:
        if (!knownArg) {
          knownArg = arg;
        }
        argOk = !isAssumedRank && rank == knownArg->Rank();
        break;
````
- **L2401 EN**: Introduces a switch dispatch label: `case Rank::coarray:`.
  **L2401 CN**: 引入一个 switch 分发标签：`case Rank::coarray:`。
- **L2402 EN**: Executes a call or declaration centered on `IsCoarray`.
  **L2402 CN**: 执行以 `IsCoarray` 为核心的调用或声明。
- **L2403 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2403 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2404 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'coarray=' argument must have corank > 0 for intrinsic '%s'"_err_en_US,`.
  **L2405 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'coarray=' argument must have corank > 0 for intrinsic '%s'"_err_en_US,`。
- **L2406 EN**: Executes a standalone statement or declaration: `name);`.
  **L2406 CN**: 执行一条独立语句或声明：`name);`。
- **L2407 EN**: Returns from the current function with `std::nullopt`.
  **L2407 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2408 EN**: Closes the current lexical scope or compound statement.
  **L2408 CN**: 结束当前词法作用域或复合语句块。
- **L2409 EN**: Exits the nearest loop or switch statement.
  **L2409 CN**: 退出最近的循环或 switch 语句。
- **L2410 EN**: Introduces a switch dispatch label: `case Rank::atom:`.
  **L2410 CN**: 引入一个 switch 分发标签：`case Rank::atom:`。
- **L2411 EN**: Executes a call or declaration centered on `&&`.
  **L2411 CN**: 执行以 `&&` 为核心的调用或声明。
- **L2412 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2412 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2413 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s=' argument must be a scalar coarray or coindexed object for intrinsic '%s'"_err_en_US,`.
  **L2414 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s=' argument must be a scalar coarray or coindexed object for intrinsic '%s'"_err_en_US,`。
- **L2415 EN**: Executes a standalone statement or declaration: `d.keyword, name);`.
  **L2415 CN**: 执行一条独立语句或声明：`d.keyword, name);`。
- **L2416 EN**: Returns from the current function with `std::nullopt`.
  **L2416 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2417 EN**: Closes the current lexical scope or compound statement.
  **L2417 CN**: 结束当前词法作用域或复合语句块。
- **L2418 EN**: Exits the nearest loop or switch statement.
  **L2418 CN**: 退出最近的循环或 switch 语句。
- **L2419 EN**: Introduces a switch dispatch label: `case Rank::known:`.
  **L2419 CN**: 引入一个 switch 分发标签：`case Rank::known:`。
- **L2420 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2420 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2421 EN**: Executes a standalone statement or declaration: `knownArg = arg;`.
  **L2421 CN**: 执行一条独立语句或声明：`knownArg = arg;`。
- **L2422 EN**: Closes the current lexical scope or compound statement.
  **L2422 CN**: 结束当前词法作用域或复合语句块。
- **L2423 EN**: Executes a call or declaration centered on `knownArg->Rank`.
  **L2423 CN**: 执行以 `knownArg->Rank` 为核心的调用或声明。
- **L2424 EN**: Exits the nearest loop or switch statement.
  **L2424 CN**: 退出最近的循环或 switch 语句。

### Lines 2425-2448

````cpp
      case Rank::anyOrAssumedRank:
      case Rank::arrayOrAssumedRank:
        if (isAssumedRank) {
          argOk = true;
          break;
        }
        if (d.rank == Rank::arrayOrAssumedRank && rank == 0) {
          argOk = false;
          break;
        }
        if (!knownArg) {
          knownArg = arg;
        }
        if (rank > 0 &&
            (std::strcmp(name, "shape") == 0 ||
                std::strcmp(name, "size") == 0 ||
                std::strcmp(name, "ubound") == 0)) {
          // Check for a whole assumed-size array argument.
          // These are disallowed for SHAPE, and require DIM= for
          // SIZE and UBOUND.
          // (A previous error message for UBOUND will take precedence
          // over this one, as this error is caught by the second entry
          // for UBOUND.)
          if (auto named{ExtractNamedEntity(*arg)}) {
````
- **L2425 EN**: Introduces a switch dispatch label: `case Rank::anyOrAssumedRank:`.
  **L2425 CN**: 引入一个 switch 分发标签：`case Rank::anyOrAssumedRank:`。
- **L2426 EN**: Introduces a switch dispatch label: `case Rank::arrayOrAssumedRank:`.
  **L2426 CN**: 引入一个 switch 分发标签：`case Rank::arrayOrAssumedRank:`。
- **L2427 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2427 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2428 EN**: Executes a standalone statement or declaration: `argOk = true;`.
  **L2428 CN**: 执行一条独立语句或声明：`argOk = true;`。
- **L2429 EN**: Exits the nearest loop or switch statement.
  **L2429 CN**: 退出最近的循环或 switch 语句。
- **L2430 EN**: Closes the current lexical scope or compound statement.
  **L2430 CN**: 结束当前词法作用域或复合语句块。
- **L2431 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2431 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2432 EN**: Executes a standalone statement or declaration: `argOk = false;`.
  **L2432 CN**: 执行一条独立语句或声明：`argOk = false;`。
- **L2433 EN**: Exits the nearest loop or switch statement.
  **L2433 CN**: 退出最近的循环或 switch 语句。
- **L2434 EN**: Closes the current lexical scope or compound statement.
  **L2434 CN**: 结束当前词法作用域或复合语句块。
- **L2435 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2435 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2436 EN**: Executes a standalone statement or declaration: `knownArg = arg;`.
  **L2436 CN**: 执行一条独立语句或声明：`knownArg = arg;`。
- **L2437 EN**: Closes the current lexical scope or compound statement.
  **L2437 CN**: 结束当前词法作用域或复合语句块。
- **L2438 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2438 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2439 EN**: Continues logic associated with callable symbol `strcmp`.
  **L2439 CN**: 继续与可调用符号 `strcmp` 相关的逻辑。
- **L2440 EN**: Continues logic associated with callable symbol `strcmp`.
  **L2440 CN**: 继续与可调用符号 `strcmp` 相关的逻辑。
- **L2441 EN**: Starts a function, method, lambda, or structured scope: `std::strcmp(name, "ubound") == 0)) {`.
  **L2441 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::strcmp(name, "ubound") == 0)) {`。
- **L2442 EN**: Comment explains nearby logic, intent, or metadata: `Check for a whole assumed-size array argument.`.
  **L2442 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check for a whole assumed-size array argument.`。
- **L2443 EN**: Comment explains nearby logic, intent, or metadata: `These are disallowed for SHAPE, and require DIM= for`.
  **L2443 CN**: 注释说明附近代码的逻辑、意图或元数据：`These are disallowed for SHAPE, and require DIM= for`。
- **L2444 EN**: Comment explains nearby logic, intent, or metadata: `SIZE and UBOUND.`.
  **L2444 CN**: 注释说明附近代码的逻辑、意图或元数据：`SIZE and UBOUND.`。
- **L2445 EN**: Comment explains nearby logic, intent, or metadata: `(A previous error message for UBOUND will take precedence`.
  **L2445 CN**: 注释说明附近代码的逻辑、意图或元数据：`(A previous error message for UBOUND will take precedence`。
- **L2446 EN**: Comment explains nearby logic, intent, or metadata: `over this one, as this error is caught by the second entry`.
  **L2446 CN**: 注释说明附近代码的逻辑、意图或元数据：`over this one, as this error is caught by the second entry`。
- **L2447 EN**: Comment explains nearby logic, intent, or metadata: `for UBOUND.)`.
  **L2447 CN**: 注释说明附近代码的逻辑、意图或元数据：`for UBOUND.)`。
- **L2448 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2448 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2449-2472

````cpp
            if (semantics::IsAssumedSizeArray(ResolveAssociations(
                    named->GetLastSymbol().GetUltimate()))) {
              if (strcmp(name, "shape") == 0) {
                messages.Say(arg->sourceLocation(),
                    "The 'source=' argument to the intrinsic function 'shape' may not be assumed-size"_err_en_US);
                return std::nullopt;
              } else if (!dimArg) {
                messages.Say(arg->sourceLocation(),
                    "A dim= argument is required for '%s' when the array is assumed-size"_err_en_US,
                    name);
                return std::nullopt;
              }
            }
          }
        }
        argOk = true;
        break;
      case Rank::conformable: // arg must be conformable with previous arrayArg
        CHECK(arrayArg);
        CHECK(arrayArgName);
        if (const std::optional<Shape> &arrayArgShape{
                GetShape(context, *arrayArg)}) {
          if (std::optional<Shape> argShape{GetShape(context, *arg)}) {
            std::string arrayArgMsg{"'"};
````
- **L2449 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2449 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2450 EN**: Starts a function, method, lambda, or structured scope: `named->GetLastSymbol().GetUltimate()))) {`.
  **L2450 CN**: 开始一个函数、方法、lambda 或结构化作用域：`named->GetLastSymbol().GetUltimate()))) {`。
- **L2451 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2451 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2452 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2453 EN**: Executes a standalone statement or declaration: `"The 'source=' argument to the intrinsic function 'shape' may not be assumed-size"_err_en_US);`.
  **L2453 CN**: 执行一条独立语句或声明：`"The 'source=' argument to the intrinsic function 'shape' may not be assumed-size"_err_en_US);`。
- **L2454 EN**: Returns from the current function with `std::nullopt`.
  **L2454 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2455 EN**: Transitions from the previous branch into an `else if` condition.
  **L2455 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2456 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"A dim= argument is required for '%s' when the array is assumed-size"_err_en_US,`.
  **L2457 CN**: 继续一个多行参数列表、初始化器或聚合项：`"A dim= argument is required for '%s' when the array is assumed-size"_err_en_US,`。
- **L2458 EN**: Executes a standalone statement or declaration: `name);`.
  **L2458 CN**: 执行一条独立语句或声明：`name);`。
- **L2459 EN**: Returns from the current function with `std::nullopt`.
  **L2459 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2460 EN**: Closes the current lexical scope or compound statement.
  **L2460 CN**: 结束当前词法作用域或复合语句块。
- **L2461 EN**: Closes the current lexical scope or compound statement.
  **L2461 CN**: 结束当前词法作用域或复合语句块。
- **L2462 EN**: Closes the current lexical scope or compound statement.
  **L2462 CN**: 结束当前词法作用域或复合语句块。
- **L2463 EN**: Closes the current lexical scope or compound statement.
  **L2463 CN**: 结束当前词法作用域或复合语句块。
- **L2464 EN**: Executes a standalone statement or declaration: `argOk = true;`.
  **L2464 CN**: 执行一条独立语句或声明：`argOk = true;`。
- **L2465 EN**: Exits the nearest loop or switch statement.
  **L2465 CN**: 退出最近的循环或 switch 语句。
- **L2466 EN**: Introduces a switch dispatch label: `case Rank::conformable: // arg must be conformable with previous arrayArg`.
  **L2466 CN**: 引入一个 switch 分发标签：`case Rank::conformable: // arg must be conformable with previous arrayArg`。
- **L2467 EN**: Executes a call or declaration centered on `CHECK`.
  **L2467 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2468 EN**: Executes a call or declaration centered on `CHECK`.
  **L2468 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2469 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2469 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2470 EN**: Starts a function, method, lambda, or structured scope: `GetShape(context, *arrayArg)}) {`.
  **L2470 CN**: 开始一个函数、方法、lambda 或结构化作用域：`GetShape(context, *arrayArg)}) {`。
- **L2471 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2471 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2472 EN**: Executes a standalone statement or declaration: `std::string arrayArgMsg{"'"};`.
  **L2472 CN**: 执行一条独立语句或声明：`std::string arrayArgMsg{"'"};`。

### Lines 2473-2496

````cpp
            arrayArgMsg = arrayArgMsg + arrayArgName + "='" + " argument";
            std::string argMsg{"'"};
            argMsg = argMsg + d.keyword + "='" + " argument";
            CheckConformance(context.messages(), *arrayArgShape, *argShape,
                CheckConformanceFlags::RightScalarExpandable,
                arrayArgMsg.c_str(), argMsg.c_str());
          }
        }
        argOk = true; // Avoid an additional error message
        break;
      case Rank::dimReduced:
      case Rank::dimRemovedOrScalar:
        CHECK(arrayArg);
        argOk = rank == 0 || rank + 1 == arrayArg->Rank();
        break;
      case Rank::reduceOperation:
        // The reduction function is validated in ApplySpecificChecks().
        argOk = true;
        break;
      case Rank::scalarIfDim:
      case Rank::locReduced:
      case Rank::rankPlus1:
      case Rank::shaped:
        common::die("INTERNAL: result-only rank code appears on argument '%s' "
````
- **L2473 EN**: Executes a standalone statement or declaration: `arrayArgMsg = arrayArgMsg + arrayArgName + "='" + " argument";`.
  **L2473 CN**: 执行一条独立语句或声明：`arrayArgMsg = arrayArgMsg + arrayArgName + "='" + " argument";`。
- **L2474 EN**: Executes a standalone statement or declaration: `std::string argMsg{"'"};`.
  **L2474 CN**: 执行一条独立语句或声明：`std::string argMsg{"'"};`。
- **L2475 EN**: Executes a standalone statement or declaration: `argMsg = argMsg + d.keyword + "='" + " argument";`.
  **L2475 CN**: 执行一条独立语句或声明：`argMsg = argMsg + d.keyword + "='" + " argument";`。
- **L2476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckConformance(context.messages(), *arrayArgShape, *argShape,`.
  **L2476 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckConformance(context.messages(), *arrayArgShape, *argShape,`。
- **L2477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckConformanceFlags::RightScalarExpandable,`.
  **L2477 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckConformanceFlags::RightScalarExpandable,`。
- **L2478 EN**: Executes a call or declaration centered on `arrayArgMsg.c_str`.
  **L2478 CN**: 执行以 `arrayArgMsg.c_str` 为核心的调用或声明。
- **L2479 EN**: Closes the current lexical scope or compound statement.
  **L2479 CN**: 结束当前词法作用域或复合语句块。
- **L2480 EN**: Closes the current lexical scope or compound statement.
  **L2480 CN**: 结束当前词法作用域或复合语句块。
- **L2481 EN**: Continues the surrounding expression or declaration: `argOk = true; // Avoid an additional error message`.
  **L2481 CN**: 继续构造周围的表达式或声明：`argOk = true; // Avoid an additional error message`。
- **L2482 EN**: Exits the nearest loop or switch statement.
  **L2482 CN**: 退出最近的循环或 switch 语句。
- **L2483 EN**: Introduces a switch dispatch label: `case Rank::dimReduced:`.
  **L2483 CN**: 引入一个 switch 分发标签：`case Rank::dimReduced:`。
- **L2484 EN**: Introduces a switch dispatch label: `case Rank::dimRemovedOrScalar:`.
  **L2484 CN**: 引入一个 switch 分发标签：`case Rank::dimRemovedOrScalar:`。
- **L2485 EN**: Executes a call or declaration centered on `CHECK`.
  **L2485 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2486 EN**: Executes a call or declaration centered on `arrayArg->Rank`.
  **L2486 CN**: 执行以 `arrayArg->Rank` 为核心的调用或声明。
- **L2487 EN**: Exits the nearest loop or switch statement.
  **L2487 CN**: 退出最近的循环或 switch 语句。
- **L2488 EN**: Introduces a switch dispatch label: `case Rank::reduceOperation:`.
  **L2488 CN**: 引入一个 switch 分发标签：`case Rank::reduceOperation:`。
- **L2489 EN**: Comment explains nearby logic, intent, or metadata: `The reduction function is validated in ApplySpecificChecks().`.
  **L2489 CN**: 注释说明附近代码的逻辑、意图或元数据：`The reduction function is validated in ApplySpecificChecks().`。
- **L2490 EN**: Executes a standalone statement or declaration: `argOk = true;`.
  **L2490 CN**: 执行一条独立语句或声明：`argOk = true;`。
- **L2491 EN**: Exits the nearest loop or switch statement.
  **L2491 CN**: 退出最近的循环或 switch 语句。
- **L2492 EN**: Introduces a switch dispatch label: `case Rank::scalarIfDim:`.
  **L2492 CN**: 引入一个 switch 分发标签：`case Rank::scalarIfDim:`。
- **L2493 EN**: Introduces a switch dispatch label: `case Rank::locReduced:`.
  **L2493 CN**: 引入一个 switch 分发标签：`case Rank::locReduced:`。
- **L2494 EN**: Introduces a switch dispatch label: `case Rank::rankPlus1:`.
  **L2494 CN**: 引入一个 switch 分发标签：`case Rank::rankPlus1:`。
- **L2495 EN**: Introduces a switch dispatch label: `case Rank::shaped:`.
  **L2495 CN**: 引入一个 switch 分发标签：`case Rank::shaped:`。
- **L2496 EN**: Continues logic associated with callable symbol `die`.
  **L2496 CN**: 继续与可调用符号 `die` 相关的逻辑。

### Lines 2497-2520

````cpp
                    "for intrinsic '%s'",
            d.keyword, name);
      }
      if (!argOk) {
        messages.Say(arg->sourceLocation(),
            "'%s=' argument has unacceptable rank %d"_err_en_US, d.keyword,
            rank);
        return std::nullopt;
      }
    }
  }

  // Calculate the characteristics of the function result, if any
  std::optional<DynamicType> resultType;
  if (auto category{result.categorySet.LeastElement()}) {
    // The intrinsic is not a subroutine.
    if (call.isSubroutineCall) {
      return std::nullopt;
    }
    switch (result.kindCode) {
    case KindCode::defaultIntegerKind:
      CHECK(result.categorySet == IntType);
      CHECK(*category == TypeCategory::Integer);
      resultType = DynamicType{TypeCategory::Integer,
````
- **L2497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"for intrinsic '%s'",`.
  **L2497 CN**: 继续一个多行参数列表、初始化器或聚合项：`"for intrinsic '%s'",`。
- **L2498 EN**: Executes a standalone statement or declaration: `d.keyword, name);`.
  **L2498 CN**: 执行一条独立语句或声明：`d.keyword, name);`。
- **L2499 EN**: Closes the current lexical scope or compound statement.
  **L2499 CN**: 结束当前词法作用域或复合语句块。
- **L2500 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2500 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg->sourceLocation(),`.
  **L2501 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg->sourceLocation(),`。
- **L2502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s=' argument has unacceptable rank %d"_err_en_US, d.keyword,`.
  **L2502 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s=' argument has unacceptable rank %d"_err_en_US, d.keyword,`。
- **L2503 EN**: Executes a standalone statement or declaration: `rank);`.
  **L2503 CN**: 执行一条独立语句或声明：`rank);`。
- **L2504 EN**: Returns from the current function with `std::nullopt`.
  **L2504 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2505 EN**: Closes the current lexical scope or compound statement.
  **L2505 CN**: 结束当前词法作用域或复合语句块。
- **L2506 EN**: Closes the current lexical scope or compound statement.
  **L2506 CN**: 结束当前词法作用域或复合语句块。
- **L2507 EN**: Closes the current lexical scope or compound statement.
  **L2507 CN**: 结束当前词法作用域或复合语句块。
- **L2508 EN**: Blank line separating nearby declarations or logic blocks.
  **L2508 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2509 EN**: Comment explains nearby logic, intent, or metadata: `Calculate the characteristics of the function result, if any`.
  **L2509 CN**: 注释说明附近代码的逻辑、意图或元数据：`Calculate the characteristics of the function result, if any`。
- **L2510 EN**: Executes a standalone statement or declaration: `std::optional<DynamicType> resultType;`.
  **L2510 CN**: 执行一条独立语句或声明：`std::optional<DynamicType> resultType;`。
- **L2511 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2511 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2512 EN**: Comment explains nearby logic, intent, or metadata: `The intrinsic is not a subroutine.`.
  **L2512 CN**: 注释说明附近代码的逻辑、意图或元数据：`The intrinsic is not a subroutine.`。
- **L2513 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2513 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2514 EN**: Returns from the current function with `std::nullopt`.
  **L2514 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2515 EN**: Closes the current lexical scope or compound statement.
  **L2515 CN**: 结束当前词法作用域或复合语句块。
- **L2516 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2516 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2517 EN**: Introduces a switch dispatch label: `case KindCode::defaultIntegerKind:`.
  **L2517 CN**: 引入一个 switch 分发标签：`case KindCode::defaultIntegerKind:`。
- **L2518 EN**: Executes a call or declaration centered on `CHECK`.
  **L2518 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2519 EN**: Executes a call or declaration centered on `CHECK`.
  **L2519 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `resultType = DynamicType{TypeCategory::Integer,`.
  **L2520 CN**: 继续一个多行参数列表、初始化器或聚合项：`resultType = DynamicType{TypeCategory::Integer,`。

### Lines 2521-2544

````cpp
          defaults.GetDefaultKind(TypeCategory::Integer)};
      break;
    case KindCode::defaultRealKind:
      CHECK(result.categorySet == CategorySet{*category});
      CHECK(FloatingType.test(*category));
      resultType =
          DynamicType{*category, defaults.GetDefaultKind(TypeCategory::Real)};
      break;
    case KindCode::doublePrecision:
      CHECK(result.categorySet == CategorySet{*category});
      CHECK(FloatingType.test(*category));
      resultType = DynamicType{*category, defaults.doublePrecisionKind()};
      break;
    case KindCode::quadPrecision:
      CHECK(result.categorySet == CategorySet{*category});
      CHECK(FloatingType.test(*category));
      resultType = DynamicType{*category, defaults.quadPrecisionKind()};
      if (!context.targetCharacteristics().CanSupportType(
              *category, defaults.quadPrecisionKind())) {
        messages.Say(
            "%s(KIND=%jd) type not supported on this target."_err_en_US,
            parser::ToUpperCaseLetters(EnumToString(*category)),
            defaults.quadPrecisionKind());
      }
````
- **L2521 EN**: Executes a call or declaration centered on `defaults.GetDefaultKind`.
  **L2521 CN**: 执行以 `defaults.GetDefaultKind` 为核心的调用或声明。
- **L2522 EN**: Exits the nearest loop or switch statement.
  **L2522 CN**: 退出最近的循环或 switch 语句。
- **L2523 EN**: Introduces a switch dispatch label: `case KindCode::defaultRealKind:`.
  **L2523 CN**: 引入一个 switch 分发标签：`case KindCode::defaultRealKind:`。
- **L2524 EN**: Executes a call or declaration centered on `CHECK`.
  **L2524 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2525 EN**: Executes a call or declaration centered on `CHECK`.
  **L2525 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2526 EN**: Continues the surrounding expression or declaration: `resultType =`.
  **L2526 CN**: 继续构造周围的表达式或声明：`resultType =`。
- **L2527 EN**: Executes a call or declaration centered on `defaults.GetDefaultKind`.
  **L2527 CN**: 执行以 `defaults.GetDefaultKind` 为核心的调用或声明。
- **L2528 EN**: Exits the nearest loop or switch statement.
  **L2528 CN**: 退出最近的循环或 switch 语句。
- **L2529 EN**: Introduces a switch dispatch label: `case KindCode::doublePrecision:`.
  **L2529 CN**: 引入一个 switch 分发标签：`case KindCode::doublePrecision:`。
- **L2530 EN**: Executes a call or declaration centered on `CHECK`.
  **L2530 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2531 EN**: Executes a call or declaration centered on `CHECK`.
  **L2531 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2532 EN**: Executes a call or declaration centered on `defaults.doublePrecisionKind`.
  **L2532 CN**: 执行以 `defaults.doublePrecisionKind` 为核心的调用或声明。
- **L2533 EN**: Exits the nearest loop or switch statement.
  **L2533 CN**: 退出最近的循环或 switch 语句。
- **L2534 EN**: Introduces a switch dispatch label: `case KindCode::quadPrecision:`.
  **L2534 CN**: 引入一个 switch 分发标签：`case KindCode::quadPrecision:`。
- **L2535 EN**: Executes a call or declaration centered on `CHECK`.
  **L2535 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2536 EN**: Executes a call or declaration centered on `CHECK`.
  **L2536 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2537 EN**: Executes a call or declaration centered on `defaults.quadPrecisionKind`.
  **L2537 CN**: 执行以 `defaults.quadPrecisionKind` 为核心的调用或声明。
- **L2538 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2538 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2539 EN**: Comment explains nearby logic, intent, or metadata: `category, defaults.quadPrecisionKind())) {`.
  **L2539 CN**: 注释说明附近代码的逻辑、意图或元数据：`category, defaults.quadPrecisionKind())) {`。
- **L2540 EN**: Continues logic associated with callable symbol `Say`.
  **L2540 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s(KIND=%jd) type not supported on this target."_err_en_US,`.
  **L2541 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s(KIND=%jd) type not supported on this target."_err_en_US,`。
- **L2542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(EnumToString(*category)),`.
  **L2542 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(EnumToString(*category)),`。
- **L2543 EN**: Executes a call or declaration centered on `defaults.quadPrecisionKind`.
  **L2543 CN**: 执行以 `defaults.quadPrecisionKind` 为核心的调用或声明。
- **L2544 EN**: Closes the current lexical scope or compound statement.
  **L2544 CN**: 结束当前词法作用域或复合语句块。

### Lines 2545-2568

````cpp
      break;
    case KindCode::defaultLogicalKind:
      CHECK(result.categorySet == LogicalType);
      CHECK(*category == TypeCategory::Logical);
      resultType = DynamicType{TypeCategory::Logical,
          defaults.GetDefaultKind(TypeCategory::Logical)};
      break;
    case KindCode::defaultCharKind:
      CHECK(result.categorySet == CharType);
      CHECK(*category == TypeCategory::Character);
      resultType = DynamicType{TypeCategory::Character,
          defaults.GetDefaultKind(TypeCategory::Character)};
      break;
    case KindCode::same:
      CHECK(sameArg);
      if (std::optional<DynamicType> aType{sameArg->GetType()}) {
        if (result.categorySet.test(aType->category())) {
          if (const auto *sameChar{UnwrapExpr<Expr<SomeCharacter>>(*sameArg)}) {
            if (auto len{ToInt64(Fold(context, sameChar->LEN()))}) {
              resultType = DynamicType{aType->kind(), *len};
            } else {
              resultType = *aType;
            }
          } else {
````
- **L2545 EN**: Exits the nearest loop or switch statement.
  **L2545 CN**: 退出最近的循环或 switch 语句。
- **L2546 EN**: Introduces a switch dispatch label: `case KindCode::defaultLogicalKind:`.
  **L2546 CN**: 引入一个 switch 分发标签：`case KindCode::defaultLogicalKind:`。
- **L2547 EN**: Executes a call or declaration centered on `CHECK`.
  **L2547 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2548 EN**: Executes a call or declaration centered on `CHECK`.
  **L2548 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `resultType = DynamicType{TypeCategory::Logical,`.
  **L2549 CN**: 继续一个多行参数列表、初始化器或聚合项：`resultType = DynamicType{TypeCategory::Logical,`。
- **L2550 EN**: Executes a call or declaration centered on `defaults.GetDefaultKind`.
  **L2550 CN**: 执行以 `defaults.GetDefaultKind` 为核心的调用或声明。
- **L2551 EN**: Exits the nearest loop or switch statement.
  **L2551 CN**: 退出最近的循环或 switch 语句。
- **L2552 EN**: Introduces a switch dispatch label: `case KindCode::defaultCharKind:`.
  **L2552 CN**: 引入一个 switch 分发标签：`case KindCode::defaultCharKind:`。
- **L2553 EN**: Executes a call or declaration centered on `CHECK`.
  **L2553 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2554 EN**: Executes a call or declaration centered on `CHECK`.
  **L2554 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `resultType = DynamicType{TypeCategory::Character,`.
  **L2555 CN**: 继续一个多行参数列表、初始化器或聚合项：`resultType = DynamicType{TypeCategory::Character,`。
- **L2556 EN**: Executes a call or declaration centered on `defaults.GetDefaultKind`.
  **L2556 CN**: 执行以 `defaults.GetDefaultKind` 为核心的调用或声明。
- **L2557 EN**: Exits the nearest loop or switch statement.
  **L2557 CN**: 退出最近的循环或 switch 语句。
- **L2558 EN**: Introduces a switch dispatch label: `case KindCode::same:`.
  **L2558 CN**: 引入一个 switch 分发标签：`case KindCode::same:`。
- **L2559 EN**: Executes a call or declaration centered on `CHECK`.
  **L2559 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2560 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2560 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2561 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2561 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2562 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2562 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2563 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2563 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2564 EN**: Executes a call or declaration centered on `DynamicType{aType->kind`.
  **L2564 CN**: 执行以 `DynamicType{aType->kind` 为核心的调用或声明。
- **L2565 EN**: Transitions from the previous branch into the alternative path.
  **L2565 CN**: 从前一个分支过渡到备选路径。
- **L2566 EN**: Executes a standalone statement or declaration: `resultType = *aType;`.
  **L2566 CN**: 执行一条独立语句或声明：`resultType = *aType;`。
- **L2567 EN**: Closes the current lexical scope or compound statement.
  **L2567 CN**: 结束当前词法作用域或复合语句块。
- **L2568 EN**: Transitions from the previous branch into the alternative path.
  **L2568 CN**: 从前一个分支过渡到备选路径。

### Lines 2569-2592

````cpp
            resultType = *aType;
          }
        } else {
          resultType = DynamicType{*category, aType->kind()};
        }
      }
      break;
    case KindCode::sameKind:
      CHECK(sameArg);
      if (std::optional<DynamicType> aType{sameArg->GetType()}) {
        resultType = DynamicType{*category, aType->kind()};
      }
      break;
    case KindCode::operand:
      CHECK(operandArg);
      resultType = operandArg->GetType();
      CHECK(!resultType || result.categorySet.test(resultType->category()));
      break;
    case KindCode::effectiveKind:
      CHECK(kindDummyArg);
      CHECK(result.categorySet == CategorySet{*category});
      if (kindArg) {
        auto *expr{kindArg->UnwrapExpr()};
        if (expr) {
````
- **L2569 EN**: Executes a standalone statement or declaration: `resultType = *aType;`.
  **L2569 CN**: 执行一条独立语句或声明：`resultType = *aType;`。
- **L2570 EN**: Closes the current lexical scope or compound statement.
  **L2570 CN**: 结束当前词法作用域或复合语句块。
- **L2571 EN**: Transitions from the previous branch into the alternative path.
  **L2571 CN**: 从前一个分支过渡到备选路径。
- **L2572 EN**: Executes a call or declaration centered on `aType->kind`.
  **L2572 CN**: 执行以 `aType->kind` 为核心的调用或声明。
- **L2573 EN**: Closes the current lexical scope or compound statement.
  **L2573 CN**: 结束当前词法作用域或复合语句块。
- **L2574 EN**: Closes the current lexical scope or compound statement.
  **L2574 CN**: 结束当前词法作用域或复合语句块。
- **L2575 EN**: Exits the nearest loop or switch statement.
  **L2575 CN**: 退出最近的循环或 switch 语句。
- **L2576 EN**: Introduces a switch dispatch label: `case KindCode::sameKind:`.
  **L2576 CN**: 引入一个 switch 分发标签：`case KindCode::sameKind:`。
- **L2577 EN**: Executes a call or declaration centered on `CHECK`.
  **L2577 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2578 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2578 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2579 EN**: Executes a call or declaration centered on `aType->kind`.
  **L2579 CN**: 执行以 `aType->kind` 为核心的调用或声明。
- **L2580 EN**: Closes the current lexical scope or compound statement.
  **L2580 CN**: 结束当前词法作用域或复合语句块。
- **L2581 EN**: Exits the nearest loop or switch statement.
  **L2581 CN**: 退出最近的循环或 switch 语句。
- **L2582 EN**: Introduces a switch dispatch label: `case KindCode::operand:`.
  **L2582 CN**: 引入一个 switch 分发标签：`case KindCode::operand:`。
- **L2583 EN**: Executes a call or declaration centered on `CHECK`.
  **L2583 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2584 EN**: Executes a call or declaration centered on `operandArg->GetType`.
  **L2584 CN**: 执行以 `operandArg->GetType` 为核心的调用或声明。
- **L2585 EN**: Executes a call or declaration centered on `CHECK`.
  **L2585 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2586 EN**: Exits the nearest loop or switch statement.
  **L2586 CN**: 退出最近的循环或 switch 语句。
- **L2587 EN**: Introduces a switch dispatch label: `case KindCode::effectiveKind:`.
  **L2587 CN**: 引入一个 switch 分发标签：`case KindCode::effectiveKind:`。
- **L2588 EN**: Executes a call or declaration centered on `CHECK`.
  **L2588 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2589 EN**: Executes a call or declaration centered on `CHECK`.
  **L2589 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2590 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2590 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2591 EN**: Executes a call or declaration centered on `*expr{kindArg->UnwrapExpr`.
  **L2591 CN**: 执行以 `*expr{kindArg->UnwrapExpr` 为核心的调用或声明。
- **L2592 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2592 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2593-2616

````cpp
          CHECK(expr->Rank() == 0);
          if (auto code{ToInt64(Fold(context, common::Clone(*expr)))}) {
            if (context.targetCharacteristics().IsTypeEnabled(
                    *category, *code)) {
              if (*category == TypeCategory::Character) { // ACHAR & CHAR
                resultType = DynamicType{static_cast<int>(*code), 1};
              } else {
                resultType = DynamicType{*category, static_cast<int>(*code)};
              }
              break;
            }
          }
        }
        if (context.analyzingPDTComponentKindSelector() && expr &&
            IsConstantExpr(*expr, &context)) {
          // Don't emit an error about a KIND= actual argument value when
          // processing a kind selector in a PDT component declaration before
          // it is instantianted, so long as it's a constant expression.
          // It will be renanalyzed later during instantiation.
        } else {
          messages.Say(
              "'kind=' argument must be a constant scalar integer whose value is a supported kind for the intrinsic result type"_err_en_US);
        }
        // use default kind below for error recovery
````
- **L2593 EN**: Executes a call or declaration centered on `CHECK`.
  **L2593 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2594 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2594 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2595 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2595 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2596 EN**: Comment explains nearby logic, intent, or metadata: `category, *code)) {`.
  **L2596 CN**: 注释说明附近代码的逻辑、意图或元数据：`category, *code)) {`。
- **L2597 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2597 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2598 EN**: Executes a call or declaration centered on `DynamicType{static_cast<int>`.
  **L2598 CN**: 执行以 `DynamicType{static_cast<int>` 为核心的调用或声明。
- **L2599 EN**: Transitions from the previous branch into the alternative path.
  **L2599 CN**: 从前一个分支过渡到备选路径。
- **L2600 EN**: Executes a call or declaration centered on `static_cast<int>`.
  **L2600 CN**: 执行以 `static_cast<int>` 为核心的调用或声明。
- **L2601 EN**: Closes the current lexical scope or compound statement.
  **L2601 CN**: 结束当前词法作用域或复合语句块。
- **L2602 EN**: Exits the nearest loop or switch statement.
  **L2602 CN**: 退出最近的循环或 switch 语句。
- **L2603 EN**: Closes the current lexical scope or compound statement.
  **L2603 CN**: 结束当前词法作用域或复合语句块。
- **L2604 EN**: Closes the current lexical scope or compound statement.
  **L2604 CN**: 结束当前词法作用域或复合语句块。
- **L2605 EN**: Closes the current lexical scope or compound statement.
  **L2605 CN**: 结束当前词法作用域或复合语句块。
- **L2606 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2606 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2607 EN**: Starts a function, method, lambda, or structured scope: `IsConstantExpr(*expr, &context)) {`.
  **L2607 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsConstantExpr(*expr, &context)) {`。
- **L2608 EN**: Comment explains nearby logic, intent, or metadata: `Don't emit an error about a KIND= actual argument value when`.
  **L2608 CN**: 注释说明附近代码的逻辑、意图或元数据：`Don't emit an error about a KIND= actual argument value when`。
- **L2609 EN**: Comment explains nearby logic, intent, or metadata: `processing a kind selector in a PDT component declaration before`.
  **L2609 CN**: 注释说明附近代码的逻辑、意图或元数据：`processing a kind selector in a PDT component declaration before`。
- **L2610 EN**: Comment explains nearby logic, intent, or metadata: `it is instantianted, so long as it's a constant expression.`.
  **L2610 CN**: 注释说明附近代码的逻辑、意图或元数据：`it is instantianted, so long as it's a constant expression.`。
- **L2611 EN**: Comment explains nearby logic, intent, or metadata: `It will be renanalyzed later during instantiation.`.
  **L2611 CN**: 注释说明附近代码的逻辑、意图或元数据：`It will be renanalyzed later during instantiation.`。
- **L2612 EN**: Transitions from the previous branch into the alternative path.
  **L2612 CN**: 从前一个分支过渡到备选路径。
- **L2613 EN**: Continues logic associated with callable symbol `Say`.
  **L2613 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2614 EN**: Executes a standalone statement or declaration: `"'kind=' argument must be a constant scalar integer whose value is a supported kind for the intrinsic result type"_err_en_US);`.
  **L2614 CN**: 执行一条独立语句或声明：`"'kind=' argument must be a constant scalar integer whose value is a supported kind for the intrinsic result type"_err_en_US);`。
- **L2615 EN**: Closes the current lexical scope or compound statement.
  **L2615 CN**: 结束当前词法作用域或复合语句块。
- **L2616 EN**: Comment explains nearby logic, intent, or metadata: `use default kind below for error recovery`.
  **L2616 CN**: 注释说明附近代码的逻辑、意图或元数据：`use default kind below for error recovery`。

### Lines 2617-2640

````cpp
      } else if (kindDummyArg->flags.test(ArgFlag::defaultsToSameKind)) {
        CHECK(sameArg);
        resultType = *sameArg->GetType();
      } else if (kindDummyArg->flags.test(ArgFlag::defaultsToSizeKind)) {
        CHECK(*category == TypeCategory::Integer);
        resultType =
            DynamicType{TypeCategory::Integer, defaults.sizeIntegerKind()};
      } else {
        CHECK(kindDummyArg->flags.test(ArgFlag::defaultsToDefaultForResult));
      }
      if (!resultType) {
        int kind{defaults.GetDefaultKind(*category)};
        if (*category == TypeCategory::Character) { // ACHAR & CHAR
          resultType = DynamicType{kind, 1};
        } else {
          resultType = DynamicType{*category, kind};
        }
      }
      break;
    case KindCode::likeMultiply:
      CHECK(dummies >= 2);
      CHECK(actualForDummy[0]);
      CHECK(actualForDummy[1]);
      resultType = actualForDummy[0]->GetType()->ResultTypeForMultiply(
````
- **L2617 EN**: Transitions from the previous branch into an `else if` condition.
  **L2617 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2618 EN**: Executes a call or declaration centered on `CHECK`.
  **L2618 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2619 EN**: Executes a call or declaration centered on `*sameArg->GetType`.
  **L2619 CN**: 执行以 `*sameArg->GetType` 为核心的调用或声明。
- **L2620 EN**: Transitions from the previous branch into an `else if` condition.
  **L2620 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2621 EN**: Executes a call or declaration centered on `CHECK`.
  **L2621 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2622 EN**: Continues the surrounding expression or declaration: `resultType =`.
  **L2622 CN**: 继续构造周围的表达式或声明：`resultType =`。
- **L2623 EN**: Executes a call or declaration centered on `defaults.sizeIntegerKind`.
  **L2623 CN**: 执行以 `defaults.sizeIntegerKind` 为核心的调用或声明。
- **L2624 EN**: Transitions from the previous branch into the alternative path.
  **L2624 CN**: 从前一个分支过渡到备选路径。
- **L2625 EN**: Executes a call or declaration centered on `CHECK`.
  **L2625 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2626 EN**: Closes the current lexical scope or compound statement.
  **L2626 CN**: 结束当前词法作用域或复合语句块。
- **L2627 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2627 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2628 EN**: Executes a call or declaration centered on `kind{defaults.GetDefaultKind`.
  **L2628 CN**: 执行以 `kind{defaults.GetDefaultKind` 为核心的调用或声明。
- **L2629 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2629 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2630 EN**: Executes a standalone statement or declaration: `resultType = DynamicType{kind, 1};`.
  **L2630 CN**: 执行一条独立语句或声明：`resultType = DynamicType{kind, 1};`。
- **L2631 EN**: Transitions from the previous branch into the alternative path.
  **L2631 CN**: 从前一个分支过渡到备选路径。
- **L2632 EN**: Executes a standalone statement or declaration: `resultType = DynamicType{*category, kind};`.
  **L2632 CN**: 执行一条独立语句或声明：`resultType = DynamicType{*category, kind};`。
- **L2633 EN**: Closes the current lexical scope or compound statement.
  **L2633 CN**: 结束当前词法作用域或复合语句块。
- **L2634 EN**: Closes the current lexical scope or compound statement.
  **L2634 CN**: 结束当前词法作用域或复合语句块。
- **L2635 EN**: Exits the nearest loop or switch statement.
  **L2635 CN**: 退出最近的循环或 switch 语句。
- **L2636 EN**: Introduces a switch dispatch label: `case KindCode::likeMultiply:`.
  **L2636 CN**: 引入一个 switch 分发标签：`case KindCode::likeMultiply:`。
- **L2637 EN**: Executes a call or declaration centered on `CHECK`.
  **L2637 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2638 EN**: Executes a call or declaration centered on `CHECK`.
  **L2638 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2639 EN**: Executes a call or declaration centered on `CHECK`.
  **L2639 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2640 EN**: Continues logic associated with callable symbol `GetType`.
  **L2640 CN**: 继续与可调用符号 `GetType` 相关的逻辑。

### Lines 2641-2664

````cpp
          *actualForDummy[1]->GetType());
      break;
    case KindCode::subscript:
      CHECK(result.categorySet == IntType);
      CHECK(*category == TypeCategory::Integer);
      resultType =
          DynamicType{TypeCategory::Integer, defaults.subscriptIntegerKind()};
      break;
    case KindCode::size:
      CHECK(result.categorySet == IntType);
      CHECK(*category == TypeCategory::Integer);
      resultType =
          DynamicType{TypeCategory::Integer, defaults.sizeIntegerKind()};
      break;
    case KindCode::teamType:
      CHECK(result.categorySet == DerivedType);
      CHECK(*category == TypeCategory::Derived);
      resultType = DynamicType{
          GetBuiltinDerivedType(builtinsScope, "__builtin_team_type")};
      break;
    case KindCode::greaterOrEqualToKind:
    case KindCode::exactKind:
      resultType = DynamicType{*category, result.kindValue};
      break;
````
- **L2641 EN**: Comment explains nearby logic, intent, or metadata: `actualForDummy[1]->GetType());`.
  **L2641 CN**: 注释说明附近代码的逻辑、意图或元数据：`actualForDummy[1]->GetType());`。
- **L2642 EN**: Exits the nearest loop or switch statement.
  **L2642 CN**: 退出最近的循环或 switch 语句。
- **L2643 EN**: Introduces a switch dispatch label: `case KindCode::subscript:`.
  **L2643 CN**: 引入一个 switch 分发标签：`case KindCode::subscript:`。
- **L2644 EN**: Executes a call or declaration centered on `CHECK`.
  **L2644 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2645 EN**: Executes a call or declaration centered on `CHECK`.
  **L2645 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2646 EN**: Continues the surrounding expression or declaration: `resultType =`.
  **L2646 CN**: 继续构造周围的表达式或声明：`resultType =`。
- **L2647 EN**: Executes a call or declaration centered on `defaults.subscriptIntegerKind`.
  **L2647 CN**: 执行以 `defaults.subscriptIntegerKind` 为核心的调用或声明。
- **L2648 EN**: Exits the nearest loop or switch statement.
  **L2648 CN**: 退出最近的循环或 switch 语句。
- **L2649 EN**: Introduces a switch dispatch label: `case KindCode::size:`.
  **L2649 CN**: 引入一个 switch 分发标签：`case KindCode::size:`。
- **L2650 EN**: Executes a call or declaration centered on `CHECK`.
  **L2650 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2651 EN**: Executes a call or declaration centered on `CHECK`.
  **L2651 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2652 EN**: Continues the surrounding expression or declaration: `resultType =`.
  **L2652 CN**: 继续构造周围的表达式或声明：`resultType =`。
- **L2653 EN**: Executes a call or declaration centered on `defaults.sizeIntegerKind`.
  **L2653 CN**: 执行以 `defaults.sizeIntegerKind` 为核心的调用或声明。
- **L2654 EN**: Exits the nearest loop or switch statement.
  **L2654 CN**: 退出最近的循环或 switch 语句。
- **L2655 EN**: Introduces a switch dispatch label: `case KindCode::teamType:`.
  **L2655 CN**: 引入一个 switch 分发标签：`case KindCode::teamType:`。
- **L2656 EN**: Executes a call or declaration centered on `CHECK`.
  **L2656 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2657 EN**: Executes a call or declaration centered on `CHECK`.
  **L2657 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2658 EN**: Continues the surrounding expression or declaration: `resultType = DynamicType{`.
  **L2658 CN**: 继续构造周围的表达式或声明：`resultType = DynamicType{`。
- **L2659 EN**: Executes a call or declaration centered on `GetBuiltinDerivedType`.
  **L2659 CN**: 执行以 `GetBuiltinDerivedType` 为核心的调用或声明。
- **L2660 EN**: Exits the nearest loop or switch statement.
  **L2660 CN**: 退出最近的循环或 switch 语句。
- **L2661 EN**: Introduces a switch dispatch label: `case KindCode::greaterOrEqualToKind:`.
  **L2661 CN**: 引入一个 switch 分发标签：`case KindCode::greaterOrEqualToKind:`。
- **L2662 EN**: Introduces a switch dispatch label: `case KindCode::exactKind:`.
  **L2662 CN**: 引入一个 switch 分发标签：`case KindCode::exactKind:`。
- **L2663 EN**: Executes a standalone statement or declaration: `resultType = DynamicType{*category, result.kindValue};`.
  **L2663 CN**: 执行一条独立语句或声明：`resultType = DynamicType{*category, result.kindValue};`。
- **L2664 EN**: Exits the nearest loop or switch statement.
  **L2664 CN**: 退出最近的循环或 switch 语句。

### Lines 2665-2688

````cpp
    case KindCode::typeless:
    case KindCode::any:
    case KindCode::kindArg:
    case KindCode::dimArg:
      common::die(
          "INTERNAL: bad KindCode appears on intrinsic '%s' result", name);
      break;
    default:
      CRASH_NO_CASE;
    }
  } else {
    if (!call.isSubroutineCall) {
      return std::nullopt;
    }
    CHECK(result.kindCode == KindCode::none);
  }

  // Emit warnings when the syntactic presence of a DIM= argument determines
  // the semantics of the call but the associated actual argument may not be
  // present at execution time.
  if (dimArg) {
    std::optional<int> arrayRank;
    if (arrayArg) {
      arrayRank = arrayArg->Rank();
````
- **L2665 EN**: Introduces a switch dispatch label: `case KindCode::typeless:`.
  **L2665 CN**: 引入一个 switch 分发标签：`case KindCode::typeless:`。
- **L2666 EN**: Introduces a switch dispatch label: `case KindCode::any:`.
  **L2666 CN**: 引入一个 switch 分发标签：`case KindCode::any:`。
- **L2667 EN**: Introduces a switch dispatch label: `case KindCode::kindArg:`.
  **L2667 CN**: 引入一个 switch 分发标签：`case KindCode::kindArg:`。
- **L2668 EN**: Introduces a switch dispatch label: `case KindCode::dimArg:`.
  **L2668 CN**: 引入一个 switch 分发标签：`case KindCode::dimArg:`。
- **L2669 EN**: Continues logic associated with callable symbol `die`.
  **L2669 CN**: 继续与可调用符号 `die` 相关的逻辑。
- **L2670 EN**: Executes a standalone statement or declaration: `"INTERNAL: bad KindCode appears on intrinsic '%s' result", name);`.
  **L2670 CN**: 执行一条独立语句或声明：`"INTERNAL: bad KindCode appears on intrinsic '%s' result", name);`。
- **L2671 EN**: Exits the nearest loop or switch statement.
  **L2671 CN**: 退出最近的循环或 switch 语句。
- **L2672 EN**: Introduces a switch dispatch label: `default:`.
  **L2672 CN**: 引入一个 switch 分发标签：`default:`。
- **L2673 EN**: Executes a standalone statement or declaration: `CRASH_NO_CASE;`.
  **L2673 CN**: 执行一条独立语句或声明：`CRASH_NO_CASE;`。
- **L2674 EN**: Closes the current lexical scope or compound statement.
  **L2674 CN**: 结束当前词法作用域或复合语句块。
- **L2675 EN**: Transitions from the previous branch into the alternative path.
  **L2675 CN**: 从前一个分支过渡到备选路径。
- **L2676 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2676 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2677 EN**: Returns from the current function with `std::nullopt`.
  **L2677 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2678 EN**: Closes the current lexical scope or compound statement.
  **L2678 CN**: 结束当前词法作用域或复合语句块。
- **L2679 EN**: Executes a call or declaration centered on `CHECK`.
  **L2679 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2680 EN**: Closes the current lexical scope or compound statement.
  **L2680 CN**: 结束当前词法作用域或复合语句块。
- **L2681 EN**: Blank line separating nearby declarations or logic blocks.
  **L2681 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2682 EN**: Comment explains nearby logic, intent, or metadata: `Emit warnings when the syntactic presence of a DIM= argument determines`.
  **L2682 CN**: 注释说明附近代码的逻辑、意图或元数据：`Emit warnings when the syntactic presence of a DIM= argument determines`。
- **L2683 EN**: Comment explains nearby logic, intent, or metadata: `the semantics of the call but the associated actual argument may not be`.
  **L2683 CN**: 注释说明附近代码的逻辑、意图或元数据：`the semantics of the call but the associated actual argument may not be`。
- **L2684 EN**: Comment explains nearby logic, intent, or metadata: `present at execution time.`.
  **L2684 CN**: 注释说明附近代码的逻辑、意图或元数据：`present at execution time.`。
- **L2685 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2685 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2686 EN**: Executes a standalone statement or declaration: `std::optional<int> arrayRank;`.
  **L2686 CN**: 执行一条独立语句或声明：`std::optional<int> arrayRank;`。
- **L2687 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2687 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2688 EN**: Executes a call or declaration centered on `arrayArg->Rank`.
  **L2688 CN**: 执行以 `arrayArg->Rank` 为核心的调用或声明。

### Lines 2689-2712

````cpp
      if (auto dimVal{ToInt64(actualForDummy[*dimArg])}) {
        if (*dimVal < 1) {
          messages.Say(
              "The value of DIM= (%jd) may not be less than 1"_err_en_US,
              static_cast<std::intmax_t>(*dimVal));
        } else if (*dimVal > *arrayRank) {
          messages.Say(
              "The value of DIM= (%jd) may not be greater than %d"_err_en_US,
              static_cast<std::intmax_t>(*dimVal), *arrayRank);
        }
      }
    }
    switch (rank) {
    case Rank::dimReduced:
    case Rank::dimRemovedOrScalar:
    case Rank::locReduced:
    case Rank::scalarIfDim:
      if (dummy[*dimArg].optionality == Optionality::required) {
        if (const Symbol *whole{
                UnwrapWholeSymbolOrComponentDataRef(actualForDummy[*dimArg])}) {
          if (IsOptional(*whole) || IsAllocatableOrObjectPointer(whole)) {
            if (rank == Rank::scalarIfDim || arrayRank.value_or(-1) == 1) {
              context.Warn(common::UsageWarning::OptionalMustBePresent,
                  "The actual argument for DIM= is optional, pointer, or allocatable, and it is assumed to be present and equal to 1 at execution time"_warn_en_US);
````
- **L2689 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2689 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2690 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2690 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2691 EN**: Continues logic associated with callable symbol `Say`.
  **L2691 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The value of DIM= (%jd) may not be less than 1"_err_en_US,`.
  **L2692 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The value of DIM= (%jd) may not be less than 1"_err_en_US,`。
- **L2693 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L2693 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L2694 EN**: Transitions from the previous branch into an `else if` condition.
  **L2694 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2695 EN**: Continues logic associated with callable symbol `Say`.
  **L2695 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The value of DIM= (%jd) may not be greater than %d"_err_en_US,`.
  **L2696 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The value of DIM= (%jd) may not be greater than %d"_err_en_US,`。
- **L2697 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L2697 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L2698 EN**: Closes the current lexical scope or compound statement.
  **L2698 CN**: 结束当前词法作用域或复合语句块。
- **L2699 EN**: Closes the current lexical scope or compound statement.
  **L2699 CN**: 结束当前词法作用域或复合语句块。
- **L2700 EN**: Closes the current lexical scope or compound statement.
  **L2700 CN**: 结束当前词法作用域或复合语句块。
- **L2701 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2701 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2702 EN**: Introduces a switch dispatch label: `case Rank::dimReduced:`.
  **L2702 CN**: 引入一个 switch 分发标签：`case Rank::dimReduced:`。
- **L2703 EN**: Introduces a switch dispatch label: `case Rank::dimRemovedOrScalar:`.
  **L2703 CN**: 引入一个 switch 分发标签：`case Rank::dimRemovedOrScalar:`。
- **L2704 EN**: Introduces a switch dispatch label: `case Rank::locReduced:`.
  **L2704 CN**: 引入一个 switch 分发标签：`case Rank::locReduced:`。
- **L2705 EN**: Introduces a switch dispatch label: `case Rank::scalarIfDim:`.
  **L2705 CN**: 引入一个 switch 分发标签：`case Rank::scalarIfDim:`。
- **L2706 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2706 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2707 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2707 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2708 EN**: Starts a function, method, lambda, or structured scope: `UnwrapWholeSymbolOrComponentDataRef(actualForDummy[*dimArg])}) {`.
  **L2708 CN**: 开始一个函数、方法、lambda 或结构化作用域：`UnwrapWholeSymbolOrComponentDataRef(actualForDummy[*dimArg])}) {`。
- **L2709 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2709 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2710 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2710 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(common::UsageWarning::OptionalMustBePresent,`.
  **L2711 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(common::UsageWarning::OptionalMustBePresent,`。
- **L2712 EN**: Executes a standalone statement or declaration: `"The actual argument for DIM= is optional, pointer, or allocatable, and it is assumed to be present and equal to 1 at execution time"_warn_en_US);`.
  **L2712 CN**: 执行一条独立语句或声明：`"The actual argument for DIM= is optional, pointer, or allocatable, and it is assumed to be present and equal to 1 at execution time"_warn_en_US);`。

### Lines 2713-2736

````cpp
            } else {
              context.Warn(common::UsageWarning::OptionalMustBePresent,
                  "The actual argument for DIM= is optional, pointer, or allocatable, and may not be absent during execution; parenthesize to silence this warning"_warn_en_US);
            }
          }
        }
      }
      break;
    default:;
    }
  }

  // At this point, the call is acceptable.
  // Determine the rank of the function result.
  int resultRank{0};
  switch (rank) {
  case Rank::elemental:
    resultRank = elementalRank;
    break;
  case Rank::scalar:
    resultRank = 0;
    break;
  case Rank::vector:
    resultRank = 1;
````
- **L2713 EN**: Transitions from the previous branch into the alternative path.
  **L2713 CN**: 从前一个分支过渡到备选路径。
- **L2714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(common::UsageWarning::OptionalMustBePresent,`.
  **L2714 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(common::UsageWarning::OptionalMustBePresent,`。
- **L2715 EN**: Executes a standalone statement or declaration: `"The actual argument for DIM= is optional, pointer, or allocatable, and may not be absent during execution; parenthesize to silence this warning"_warn_en_US);`.
  **L2715 CN**: 执行一条独立语句或声明：`"The actual argument for DIM= is optional, pointer, or allocatable, and may not be absent during execution; parenthesize to silence this warning"_warn_en_US);`。
- **L2716 EN**: Closes the current lexical scope or compound statement.
  **L2716 CN**: 结束当前词法作用域或复合语句块。
- **L2717 EN**: Closes the current lexical scope or compound statement.
  **L2717 CN**: 结束当前词法作用域或复合语句块。
- **L2718 EN**: Closes the current lexical scope or compound statement.
  **L2718 CN**: 结束当前词法作用域或复合语句块。
- **L2719 EN**: Closes the current lexical scope or compound statement.
  **L2719 CN**: 结束当前词法作用域或复合语句块。
- **L2720 EN**: Exits the nearest loop or switch statement.
  **L2720 CN**: 退出最近的循环或 switch 语句。
- **L2721 EN**: Introduces a switch dispatch label: `default:;`.
  **L2721 CN**: 引入一个 switch 分发标签：`default:;`。
- **L2722 EN**: Closes the current lexical scope or compound statement.
  **L2722 CN**: 结束当前词法作用域或复合语句块。
- **L2723 EN**: Closes the current lexical scope or compound statement.
  **L2723 CN**: 结束当前词法作用域或复合语句块。
- **L2724 EN**: Blank line separating nearby declarations or logic blocks.
  **L2724 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2725 EN**: Comment explains nearby logic, intent, or metadata: `At this point, the call is acceptable.`.
  **L2725 CN**: 注释说明附近代码的逻辑、意图或元数据：`At this point, the call is acceptable.`。
- **L2726 EN**: Comment explains nearby logic, intent, or metadata: `Determine the rank of the function result.`.
  **L2726 CN**: 注释说明附近代码的逻辑、意图或元数据：`Determine the rank of the function result.`。
- **L2727 EN**: Executes a standalone statement or declaration: `int resultRank{0};`.
  **L2727 CN**: 执行一条独立语句或声明：`int resultRank{0};`。
- **L2728 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2728 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2729 EN**: Introduces a switch dispatch label: `case Rank::elemental:`.
  **L2729 CN**: 引入一个 switch 分发标签：`case Rank::elemental:`。
- **L2730 EN**: Executes a standalone statement or declaration: `resultRank = elementalRank;`.
  **L2730 CN**: 执行一条独立语句或声明：`resultRank = elementalRank;`。
- **L2731 EN**: Exits the nearest loop or switch statement.
  **L2731 CN**: 退出最近的循环或 switch 语句。
- **L2732 EN**: Introduces a switch dispatch label: `case Rank::scalar:`.
  **L2732 CN**: 引入一个 switch 分发标签：`case Rank::scalar:`。
- **L2733 EN**: Executes a standalone statement or declaration: `resultRank = 0;`.
  **L2733 CN**: 执行一条独立语句或声明：`resultRank = 0;`。
- **L2734 EN**: Exits the nearest loop or switch statement.
  **L2734 CN**: 退出最近的循环或 switch 语句。
- **L2735 EN**: Introduces a switch dispatch label: `case Rank::vector:`.
  **L2735 CN**: 引入一个 switch 分发标签：`case Rank::vector:`。
- **L2736 EN**: Executes a standalone statement or declaration: `resultRank = 1;`.
  **L2736 CN**: 执行一条独立语句或声明：`resultRank = 1;`。

### Lines 2737-2760

````cpp
    break;
  case Rank::matrix:
    resultRank = 2;
    break;
  case Rank::conformable:
    CHECK(arrayArg);
    resultRank = arrayArg->Rank();
    break;
  case Rank::dimReduced:
    CHECK(arrayArg);
    resultRank = dimArg ? arrayArg->Rank() - 1 : 0;
    break;
  case Rank::locReduced:
    CHECK(arrayArg);
    resultRank = dimArg ? arrayArg->Rank() - 1 : 1;
    break;
  case Rank::rankPlus1:
    CHECK(knownArg);
    resultRank = knownArg->Rank() + 1;
    break;
  case Rank::shaped:
    CHECK(shapeArgSize);
    resultRank = *shapeArgSize;
    break;
````
- **L2737 EN**: Exits the nearest loop or switch statement.
  **L2737 CN**: 退出最近的循环或 switch 语句。
- **L2738 EN**: Introduces a switch dispatch label: `case Rank::matrix:`.
  **L2738 CN**: 引入一个 switch 分发标签：`case Rank::matrix:`。
- **L2739 EN**: Executes a standalone statement or declaration: `resultRank = 2;`.
  **L2739 CN**: 执行一条独立语句或声明：`resultRank = 2;`。
- **L2740 EN**: Exits the nearest loop or switch statement.
  **L2740 CN**: 退出最近的循环或 switch 语句。
- **L2741 EN**: Introduces a switch dispatch label: `case Rank::conformable:`.
  **L2741 CN**: 引入一个 switch 分发标签：`case Rank::conformable:`。
- **L2742 EN**: Executes a call or declaration centered on `CHECK`.
  **L2742 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2743 EN**: Executes a call or declaration centered on `arrayArg->Rank`.
  **L2743 CN**: 执行以 `arrayArg->Rank` 为核心的调用或声明。
- **L2744 EN**: Exits the nearest loop or switch statement.
  **L2744 CN**: 退出最近的循环或 switch 语句。
- **L2745 EN**: Introduces a switch dispatch label: `case Rank::dimReduced:`.
  **L2745 CN**: 引入一个 switch 分发标签：`case Rank::dimReduced:`。
- **L2746 EN**: Executes a call or declaration centered on `CHECK`.
  **L2746 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2747 EN**: Executes a call or declaration centered on `arrayArg->Rank`.
  **L2747 CN**: 执行以 `arrayArg->Rank` 为核心的调用或声明。
- **L2748 EN**: Exits the nearest loop or switch statement.
  **L2748 CN**: 退出最近的循环或 switch 语句。
- **L2749 EN**: Introduces a switch dispatch label: `case Rank::locReduced:`.
  **L2749 CN**: 引入一个 switch 分发标签：`case Rank::locReduced:`。
- **L2750 EN**: Executes a call or declaration centered on `CHECK`.
  **L2750 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2751 EN**: Executes a call or declaration centered on `arrayArg->Rank`.
  **L2751 CN**: 执行以 `arrayArg->Rank` 为核心的调用或声明。
- **L2752 EN**: Exits the nearest loop or switch statement.
  **L2752 CN**: 退出最近的循环或 switch 语句。
- **L2753 EN**: Introduces a switch dispatch label: `case Rank::rankPlus1:`.
  **L2753 CN**: 引入一个 switch 分发标签：`case Rank::rankPlus1:`。
- **L2754 EN**: Executes a call or declaration centered on `CHECK`.
  **L2754 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2755 EN**: Executes a call or declaration centered on `knownArg->Rank`.
  **L2755 CN**: 执行以 `knownArg->Rank` 为核心的调用或声明。
- **L2756 EN**: Exits the nearest loop or switch statement.
  **L2756 CN**: 退出最近的循环或 switch 语句。
- **L2757 EN**: Introduces a switch dispatch label: `case Rank::shaped:`.
  **L2757 CN**: 引入一个 switch 分发标签：`case Rank::shaped:`。
- **L2758 EN**: Executes a call or declaration centered on `CHECK`.
  **L2758 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2759 EN**: Executes a standalone statement or declaration: `resultRank = *shapeArgSize;`.
  **L2759 CN**: 执行一条独立语句或声明：`resultRank = *shapeArgSize;`。
- **L2760 EN**: Exits the nearest loop or switch statement.
  **L2760 CN**: 退出最近的循环或 switch 语句。

### Lines 2761-2784

````cpp
  case Rank::scalarIfDim:
    resultRank = dimArg ? 0 : 1;
    break;
  case Rank::elementalOrBOZ:
  case Rank::shape:
  case Rank::array:
  case Rank::coarray:
  case Rank::atom:
  case Rank::known:
  case Rank::anyOrAssumedRank:
  case Rank::arrayOrAssumedRank:
  case Rank::reduceOperation:
  case Rank::dimRemovedOrScalar:
    common::die("INTERNAL: bad Rank code on intrinsic '%s' result", name);
    break;
  }
  CHECK(resultRank >= 0);

  // Rearrange the actual arguments into dummy argument order.
  ActualArguments rearranged(dummies);
  for (std::size_t j{0}; j < dummies; ++j) {
    if (ActualArgument *arg{actualForDummy[j]}) {
      rearranged[j] = std::move(*arg);
    }
````
- **L2761 EN**: Introduces a switch dispatch label: `case Rank::scalarIfDim:`.
  **L2761 CN**: 引入一个 switch 分发标签：`case Rank::scalarIfDim:`。
- **L2762 EN**: Executes a standalone statement or declaration: `resultRank = dimArg ? 0 : 1;`.
  **L2762 CN**: 执行一条独立语句或声明：`resultRank = dimArg ? 0 : 1;`。
- **L2763 EN**: Exits the nearest loop or switch statement.
  **L2763 CN**: 退出最近的循环或 switch 语句。
- **L2764 EN**: Introduces a switch dispatch label: `case Rank::elementalOrBOZ:`.
  **L2764 CN**: 引入一个 switch 分发标签：`case Rank::elementalOrBOZ:`。
- **L2765 EN**: Introduces a switch dispatch label: `case Rank::shape:`.
  **L2765 CN**: 引入一个 switch 分发标签：`case Rank::shape:`。
- **L2766 EN**: Introduces a switch dispatch label: `case Rank::array:`.
  **L2766 CN**: 引入一个 switch 分发标签：`case Rank::array:`。
- **L2767 EN**: Introduces a switch dispatch label: `case Rank::coarray:`.
  **L2767 CN**: 引入一个 switch 分发标签：`case Rank::coarray:`。
- **L2768 EN**: Introduces a switch dispatch label: `case Rank::atom:`.
  **L2768 CN**: 引入一个 switch 分发标签：`case Rank::atom:`。
- **L2769 EN**: Introduces a switch dispatch label: `case Rank::known:`.
  **L2769 CN**: 引入一个 switch 分发标签：`case Rank::known:`。
- **L2770 EN**: Introduces a switch dispatch label: `case Rank::anyOrAssumedRank:`.
  **L2770 CN**: 引入一个 switch 分发标签：`case Rank::anyOrAssumedRank:`。
- **L2771 EN**: Introduces a switch dispatch label: `case Rank::arrayOrAssumedRank:`.
  **L2771 CN**: 引入一个 switch 分发标签：`case Rank::arrayOrAssumedRank:`。
- **L2772 EN**: Introduces a switch dispatch label: `case Rank::reduceOperation:`.
  **L2772 CN**: 引入一个 switch 分发标签：`case Rank::reduceOperation:`。
- **L2773 EN**: Introduces a switch dispatch label: `case Rank::dimRemovedOrScalar:`.
  **L2773 CN**: 引入一个 switch 分发标签：`case Rank::dimRemovedOrScalar:`。
- **L2774 EN**: Executes a call or declaration centered on `common::die`.
  **L2774 CN**: 执行以 `common::die` 为核心的调用或声明。
- **L2775 EN**: Exits the nearest loop or switch statement.
  **L2775 CN**: 退出最近的循环或 switch 语句。
- **L2776 EN**: Closes the current lexical scope or compound statement.
  **L2776 CN**: 结束当前词法作用域或复合语句块。
- **L2777 EN**: Executes a call or declaration centered on `CHECK`.
  **L2777 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2778 EN**: Blank line separating nearby declarations or logic blocks.
  **L2778 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2779 EN**: Comment explains nearby logic, intent, or metadata: `Rearrange the actual arguments into dummy argument order.`.
  **L2779 CN**: 注释说明附近代码的逻辑、意图或元数据：`Rearrange the actual arguments into dummy argument order.`。
- **L2780 EN**: Executes a call or declaration centered on `rearranged`.
  **L2780 CN**: 执行以 `rearranged` 为核心的调用或声明。
- **L2781 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2781 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2782 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2782 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2783 EN**: Executes a call or declaration centered on `std::move`.
  **L2783 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L2784 EN**: Closes the current lexical scope or compound statement.
  **L2784 CN**: 结束当前词法作用域或复合语句块。

### Lines 2785-2808

````cpp
  }

  // Characterize the specific intrinsic procedure.
  characteristics::DummyArguments dummyArgs;
  std::optional<int> sameDummyArg;

  for (std::size_t j{0}; j < dummies; ++j) {
    const IntrinsicDummyArgument &d{dummy[std::min(j, dummyArgPatterns - 1)]};
    if (const auto &arg{rearranged[j]}) {
      if (const Expr<SomeType> *expr{arg->UnwrapExpr()}) {
        std::string kw{d.keyword};
        if (arg->keyword()) {
          kw = arg->keyword()->ToString();
        } else if (isMaxMin) {
          for (std::size_t k{j + 1};; ++k) {
            kw = "a"s + std::to_string(k);
            auto iter{std::find_if(dummyArgs.begin(), dummyArgs.end(),
                [&kw](const characteristics::DummyArgument &prev) {
                  return prev.name == kw;
                })};
            if (iter == dummyArgs.end()) {
              break;
            }
          }
````
- **L2785 EN**: Closes the current lexical scope or compound statement.
  **L2785 CN**: 结束当前词法作用域或复合语句块。
- **L2786 EN**: Blank line separating nearby declarations or logic blocks.
  **L2786 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2787 EN**: Comment explains nearby logic, intent, or metadata: `Characterize the specific intrinsic procedure.`.
  **L2787 CN**: 注释说明附近代码的逻辑、意图或元数据：`Characterize the specific intrinsic procedure.`。
- **L2788 EN**: Executes a standalone statement or declaration: `characteristics::DummyArguments dummyArgs;`.
  **L2788 CN**: 执行一条独立语句或声明：`characteristics::DummyArguments dummyArgs;`。
- **L2789 EN**: Executes a standalone statement or declaration: `std::optional<int> sameDummyArg;`.
  **L2789 CN**: 执行一条独立语句或声明：`std::optional<int> sameDummyArg;`。
- **L2790 EN**: Blank line separating nearby declarations or logic blocks.
  **L2790 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2791 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2791 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2792 EN**: Executes a call or declaration centered on `&d{dummy[std::min`.
  **L2792 CN**: 执行以 `&d{dummy[std::min` 为核心的调用或声明。
- **L2793 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2793 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2794 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2794 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2795 EN**: Executes a standalone statement or declaration: `std::string kw{d.keyword};`.
  **L2795 CN**: 执行一条独立语句或声明：`std::string kw{d.keyword};`。
- **L2796 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2796 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2797 EN**: Executes a call or declaration centered on `arg->keyword`.
  **L2797 CN**: 执行以 `arg->keyword` 为核心的调用或声明。
- **L2798 EN**: Transitions from the previous branch into an `else if` condition.
  **L2798 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2799 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2799 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2800 EN**: Executes a call or declaration centered on `std::to_string`.
  **L2800 CN**: 执行以 `std::to_string` 为核心的调用或声明。
- **L2801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto iter{std::find_if(dummyArgs.begin(), dummyArgs.end(),`.
  **L2801 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto iter{std::find_if(dummyArgs.begin(), dummyArgs.end(),`。
- **L2802 EN**: Starts a function, method, lambda, or structured scope: `[&kw](const characteristics::DummyArgument &prev) {`.
  **L2802 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&kw](const characteristics::DummyArgument &prev) {`。
- **L2803 EN**: Returns from the current function with `prev.name == kw`.
  **L2803 CN**: 以 `prev.name == kw` 从当前函数返回。
- **L2804 EN**: Executes a standalone statement or declaration: `})};`.
  **L2804 CN**: 执行一条独立语句或声明：`})};`。
- **L2805 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2805 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2806 EN**: Exits the nearest loop or switch statement.
  **L2806 CN**: 退出最近的循环或 switch 语句。
- **L2807 EN**: Closes the current lexical scope or compound statement.
  **L2807 CN**: 结束当前词法作用域或复合语句块。
- **L2808 EN**: Closes the current lexical scope or compound statement.
  **L2808 CN**: 结束当前词法作用域或复合语句块。

### Lines 2809-2832

````cpp
        }
        if (auto dc{characteristics::DummyArgument::FromActual(std::move(kw),
                *expr, context, /*forImplicitInterface=*/false)}) {
          if (auto *dummyProc{
                  std::get_if<characteristics::DummyProcedure>(&dc->u)}) {
            // Dummy procedures are never elemental.
            dummyProc->procedure.value().attrs.reset(
                characteristics::Procedure::Attr::Elemental);
          } else if (auto *dummyObject{
                         std::get_if<characteristics::DummyDataObject>(
                             &dc->u)}) {
            dummyObject->type.set_corank(0);
            if (d.flags.test(ArgFlag::onlyConstantInquiry)) {
              dummyObject->attrs.set(
                  characteristics::DummyDataObject::Attr::OnlyIntrinsicInquiry);
            }
          }
          dummyArgs.emplace_back(std::move(*dc));
          if (d.typePattern.kindCode == KindCode::same && !sameDummyArg) {
            sameDummyArg = j;
          }
        } else { // error recovery
          messages.Say(
              "Could not characterize intrinsic function actual argument '%s'"_err_en_US,
````
- **L2809 EN**: Closes the current lexical scope or compound statement.
  **L2809 CN**: 结束当前词法作用域或复合语句块。
- **L2810 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2810 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2811 EN**: Comment explains nearby logic, intent, or metadata: `expr, context, /*forImplicitInterface=*/false)}) {`.
  **L2811 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr, context, /*forImplicitInterface=*/false)}) {`。
- **L2812 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2812 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2813 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<characteristics::DummyProcedure>(&dc->u)}) {`.
  **L2813 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<characteristics::DummyProcedure>(&dc->u)}) {`。
- **L2814 EN**: Comment explains nearby logic, intent, or metadata: `Dummy procedures are never elemental.`.
  **L2814 CN**: 注释说明附近代码的逻辑、意图或元数据：`Dummy procedures are never elemental.`。
- **L2815 EN**: Continues logic associated with callable symbol `value`.
  **L2815 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L2816 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attr::Elemental);`.
  **L2816 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attr::Elemental);`。
- **L2817 EN**: Transitions from the previous branch into an `else if` condition.
  **L2817 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2818 EN**: Continues logic associated with callable symbol `DummyDataObject>`.
  **L2818 CN**: 继续与可调用符号 `DummyDataObject>` 相关的逻辑。
- **L2819 EN**: Continues the surrounding expression or declaration: `&dc->u)}) {`.
  **L2819 CN**: 继续构造周围的表达式或声明：`&dc->u)}) {`。
- **L2820 EN**: Executes a call or declaration centered on `dummyObject->type.set_corank`.
  **L2820 CN**: 执行以 `dummyObject->type.set_corank` 为核心的调用或声明。
- **L2821 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2821 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2822 EN**: Continues logic associated with callable symbol `set`.
  **L2822 CN**: 继续与可调用符号 `set` 相关的逻辑。
- **L2823 EN**: Executes a standalone statement or declaration: `characteristics::DummyDataObject::Attr::OnlyIntrinsicInquiry);`.
  **L2823 CN**: 执行一条独立语句或声明：`characteristics::DummyDataObject::Attr::OnlyIntrinsicInquiry);`。
- **L2824 EN**: Closes the current lexical scope or compound statement.
  **L2824 CN**: 结束当前词法作用域或复合语句块。
- **L2825 EN**: Closes the current lexical scope or compound statement.
  **L2825 CN**: 结束当前词法作用域或复合语句块。
- **L2826 EN**: Executes a call or declaration centered on `dummyArgs.emplace_back`.
  **L2826 CN**: 执行以 `dummyArgs.emplace_back` 为核心的调用或声明。
- **L2827 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2827 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2828 EN**: Executes a standalone statement or declaration: `sameDummyArg = j;`.
  **L2828 CN**: 执行一条独立语句或声明：`sameDummyArg = j;`。
- **L2829 EN**: Closes the current lexical scope or compound statement.
  **L2829 CN**: 结束当前词法作用域或复合语句块。
- **L2830 EN**: Transitions from the previous branch into the alternative path.
  **L2830 CN**: 从前一个分支过渡到备选路径。
- **L2831 EN**: Continues logic associated with callable symbol `Say`.
  **L2831 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Could not characterize intrinsic function actual argument '%s'"_err_en_US,`.
  **L2832 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Could not characterize intrinsic function actual argument '%s'"_err_en_US,`。

### Lines 2833-2856

````cpp
              expr->AsFortran().c_str());
          return std::nullopt;
        }
      } else {
        CHECK(arg->GetAssumedTypeDummy());
        dummyArgs.emplace_back(std::string{d.keyword},
            characteristics::DummyDataObject{DynamicType::AssumedType()});
      }
    } else {
      // optional argument is absent
      CHECK(d.optionality != Optionality::required);
      if (d.typePattern.kindCode == KindCode::same) {
        dummyArgs.emplace_back(dummyArgs[sameDummyArg.value()]);
      } else {
        auto category{d.typePattern.categorySet.LeastElement().value()};
        if (category == TypeCategory::Derived) {
          // TODO: any other built-in derived types used as optional intrinsic
          // dummies?
          CHECK(d.typePattern.kindCode == KindCode::teamType);
          characteristics::TypeAndShape typeAndShape{
              GetBuiltinDerivedType(builtinsScope, "__builtin_team_type")};
          dummyArgs.emplace_back(std::string{d.keyword},
              characteristics::DummyDataObject{std::move(typeAndShape)});
        } else {
````
- **L2833 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L2833 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L2834 EN**: Returns from the current function with `std::nullopt`.
  **L2834 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2835 EN**: Closes the current lexical scope or compound statement.
  **L2835 CN**: 结束当前词法作用域或复合语句块。
- **L2836 EN**: Transitions from the previous branch into the alternative path.
  **L2836 CN**: 从前一个分支过渡到备选路径。
- **L2837 EN**: Executes a call or declaration centered on `CHECK`.
  **L2837 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummyArgs.emplace_back(std::string{d.keyword},`.
  **L2838 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummyArgs.emplace_back(std::string{d.keyword},`。
- **L2839 EN**: Executes a call or declaration centered on `characteristics::DummyDataObject{DynamicType::AssumedType`.
  **L2839 CN**: 执行以 `characteristics::DummyDataObject{DynamicType::AssumedType` 为核心的调用或声明。
- **L2840 EN**: Closes the current lexical scope or compound statement.
  **L2840 CN**: 结束当前词法作用域或复合语句块。
- **L2841 EN**: Transitions from the previous branch into the alternative path.
  **L2841 CN**: 从前一个分支过渡到备选路径。
- **L2842 EN**: Comment explains nearby logic, intent, or metadata: `optional argument is absent`.
  **L2842 CN**: 注释说明附近代码的逻辑、意图或元数据：`optional argument is absent`。
- **L2843 EN**: Executes a call or declaration centered on `CHECK`.
  **L2843 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2844 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2844 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2845 EN**: Executes a call or declaration centered on `dummyArgs.emplace_back`.
  **L2845 CN**: 执行以 `dummyArgs.emplace_back` 为核心的调用或声明。
- **L2846 EN**: Transitions from the previous branch into the alternative path.
  **L2846 CN**: 从前一个分支过渡到备选路径。
- **L2847 EN**: Executes a call or declaration centered on `category{d.typePattern.categorySet.LeastElement`.
  **L2847 CN**: 执行以 `category{d.typePattern.categorySet.LeastElement` 为核心的调用或声明。
- **L2848 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2848 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2849 EN**: Comment records a pending task or caution: `TODO: any other built-in derived types used as optional intrinsic`.
  **L2849 CN**: 注释记录待办事项或注意点：`TODO: any other built-in derived types used as optional intrinsic`。
- **L2850 EN**: Comment explains nearby logic, intent, or metadata: `dummies?`.
  **L2850 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummies?`。
- **L2851 EN**: Executes a call or declaration centered on `CHECK`.
  **L2851 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2852 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape typeAndShape{`.
  **L2852 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape typeAndShape{`。
- **L2853 EN**: Executes a call or declaration centered on `GetBuiltinDerivedType`.
  **L2853 CN**: 执行以 `GetBuiltinDerivedType` 为核心的调用或声明。
- **L2854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummyArgs.emplace_back(std::string{d.keyword},`.
  **L2854 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummyArgs.emplace_back(std::string{d.keyword},`。
- **L2855 EN**: Executes a call or declaration centered on `characteristics::DummyDataObject{std::move`.
  **L2855 CN**: 执行以 `characteristics::DummyDataObject{std::move` 为核心的调用或声明。
- **L2856 EN**: Transitions from the previous branch into the alternative path.
  **L2856 CN**: 从前一个分支过渡到备选路径。

### Lines 2857-2880

````cpp
          characteristics::TypeAndShape typeAndShape{
              DynamicType{category, defaults.GetDefaultKind(category)}};
          dummyArgs.emplace_back(std::string{d.keyword},
              characteristics::DummyDataObject{std::move(typeAndShape)});
        }
      }
      dummyArgs.back().SetOptional();
    }
    dummyArgs.back().SetIntent(d.intent);
  }
  characteristics::Procedure::Attrs attrs;
  if (elementalRank > 0) {
    attrs.set(characteristics::Procedure::Attr::Elemental);
  }
  // TODO: Mark intrinsic procedures that are SIMPLE per F2023
  if (call.isSubroutineCall) {
    if (intrinsicClass == IntrinsicClass::pureSubroutine /* MOVE_ALLOC */ ||
        intrinsicClass == IntrinsicClass::elementalSubroutine /* MVBITS */) {
      attrs.set(characteristics::Procedure::Attr::Pure);
    }
    return SpecificCall{
        SpecificIntrinsic{
            name, characteristics::Procedure{std::move(dummyArgs), attrs}},
        std::move(rearranged)};
````
- **L2857 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape typeAndShape{`.
  **L2857 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape typeAndShape{`。
- **L2858 EN**: Executes a call or declaration centered on `defaults.GetDefaultKind`.
  **L2858 CN**: 执行以 `defaults.GetDefaultKind` 为核心的调用或声明。
- **L2859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummyArgs.emplace_back(std::string{d.keyword},`.
  **L2859 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummyArgs.emplace_back(std::string{d.keyword},`。
- **L2860 EN**: Executes a call or declaration centered on `characteristics::DummyDataObject{std::move`.
  **L2860 CN**: 执行以 `characteristics::DummyDataObject{std::move` 为核心的调用或声明。
- **L2861 EN**: Closes the current lexical scope or compound statement.
  **L2861 CN**: 结束当前词法作用域或复合语句块。
- **L2862 EN**: Closes the current lexical scope or compound statement.
  **L2862 CN**: 结束当前词法作用域或复合语句块。
- **L2863 EN**: Executes a call or declaration centered on `dummyArgs.back`.
  **L2863 CN**: 执行以 `dummyArgs.back` 为核心的调用或声明。
- **L2864 EN**: Closes the current lexical scope or compound statement.
  **L2864 CN**: 结束当前词法作用域或复合语句块。
- **L2865 EN**: Executes a call or declaration centered on `dummyArgs.back`.
  **L2865 CN**: 执行以 `dummyArgs.back` 为核心的调用或声明。
- **L2866 EN**: Closes the current lexical scope or compound statement.
  **L2866 CN**: 结束当前词法作用域或复合语句块。
- **L2867 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attrs attrs;`.
  **L2867 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attrs attrs;`。
- **L2868 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2868 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2869 EN**: Executes a call or declaration centered on `attrs.set`.
  **L2869 CN**: 执行以 `attrs.set` 为核心的调用或声明。
- **L2870 EN**: Closes the current lexical scope or compound statement.
  **L2870 CN**: 结束当前词法作用域或复合语句块。
- **L2871 EN**: Comment records a pending task or caution: `TODO: Mark intrinsic procedures that are SIMPLE per F2023`.
  **L2871 CN**: 注释记录待办事项或注意点：`TODO: Mark intrinsic procedures that are SIMPLE per F2023`。
- **L2872 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2872 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2873 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2873 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2874 EN**: Continues the surrounding expression or declaration: `intrinsicClass == IntrinsicClass::elementalSubroutine /* MVBITS */) {`.
  **L2874 CN**: 继续构造周围的表达式或声明：`intrinsicClass == IntrinsicClass::elementalSubroutine /* MVBITS */) {`。
- **L2875 EN**: Executes a call or declaration centered on `attrs.set`.
  **L2875 CN**: 执行以 `attrs.set` 为核心的调用或声明。
- **L2876 EN**: Closes the current lexical scope or compound statement.
  **L2876 CN**: 结束当前词法作用域或复合语句块。
- **L2877 EN**: Returns from the current function with `SpecificCall{`.
  **L2877 CN**: 以 `SpecificCall{` 从当前函数返回。
- **L2878 EN**: Continues the surrounding expression or declaration: `SpecificIntrinsic{`.
  **L2878 CN**: 继续构造周围的表达式或声明：`SpecificIntrinsic{`。
- **L2879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `name, characteristics::Procedure{std::move(dummyArgs), attrs}},`.
  **L2879 CN**: 继续一个多行参数列表、初始化器或聚合项：`name, characteristics::Procedure{std::move(dummyArgs), attrs}},`。
- **L2880 EN**: Executes a call or declaration centered on `std::move`.
  **L2880 CN**: 执行以 `std::move` 为核心的调用或声明。

### Lines 2881-2904

````cpp
  } else {
    if (intrinsicClass != IntrinsicClass::impureFunction /* RAND and IRAND */)
      attrs.set(characteristics::Procedure::Attr::Pure);
    characteristics::TypeAndShape typeAndShape{resultType.value(), resultRank};
    characteristics::FunctionResult funcResult{std::move(typeAndShape)};
    characteristics::Procedure chars{
        std::move(funcResult), std::move(dummyArgs), attrs};
    return SpecificCall{
        SpecificIntrinsic{name, std::move(chars)}, std::move(rearranged)};
  }
}

class IntrinsicProcTable::Implementation {
public:
  explicit Implementation(const common::IntrinsicTypeDefaultKinds &dfts)
      : defaults_{dfts} {
    for (const IntrinsicInterface &f : genericIntrinsicFunction) {
      genericFuncs_.insert(std::make_pair(std::string{f.name}, &f));
    }
    for (const std::pair<const char *, const char *> &a : genericAlias) {
      aliases_.insert(
          std::make_pair(std::string{a.first}, std::string{a.second}));
    }
    for (const SpecificIntrinsicInterface &f : specificIntrinsicFunction) {
````
- **L2881 EN**: Transitions from the previous branch into the alternative path.
  **L2881 CN**: 从前一个分支过渡到备选路径。
- **L2882 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2882 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2883 EN**: Executes a call or declaration centered on `attrs.set`.
  **L2883 CN**: 执行以 `attrs.set` 为核心的调用或声明。
- **L2884 EN**: Executes a call or declaration centered on `typeAndShape{resultType.value`.
  **L2884 CN**: 执行以 `typeAndShape{resultType.value` 为核心的调用或声明。
- **L2885 EN**: Executes a call or declaration centered on `funcResult{std::move`.
  **L2885 CN**: 执行以 `funcResult{std::move` 为核心的调用或声明。
- **L2886 EN**: Continues the surrounding expression or declaration: `characteristics::Procedure chars{`.
  **L2886 CN**: 继续构造周围的表达式或声明：`characteristics::Procedure chars{`。
- **L2887 EN**: Executes a call or declaration centered on `std::move`.
  **L2887 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L2888 EN**: Returns from the current function with `SpecificCall{`.
  **L2888 CN**: 以 `SpecificCall{` 从当前函数返回。
- **L2889 EN**: Executes a call or declaration centered on `std::move`.
  **L2889 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L2890 EN**: Closes the current lexical scope or compound statement.
  **L2890 CN**: 结束当前词法作用域或复合语句块。
- **L2891 EN**: Closes the current lexical scope or compound statement.
  **L2891 CN**: 结束当前词法作用域或复合语句块。
- **L2892 EN**: Blank line separating nearby declarations or logic blocks.
  **L2892 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2893 EN**: Declares class `IntrinsicProcTable`.
  **L2893 CN**: 声明 class `IntrinsicProcTable`。
- **L2894 EN**: Sets the following members to `public` access.
  **L2894 CN**: 将后续成员的访问级别设为 `public`。
- **L2895 EN**: Continues logic associated with callable symbol `Implementation`.
  **L2895 CN**: 继续与可调用符号 `Implementation` 相关的逻辑。
- **L2896 EN**: Continues the surrounding expression or declaration: `: defaults_{dfts} {`.
  **L2896 CN**: 继续构造周围的表达式或声明：`: defaults_{dfts} {`。
- **L2897 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2897 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2898 EN**: Executes a call or declaration centered on `genericFuncs_.insert`.
  **L2898 CN**: 执行以 `genericFuncs_.insert` 为核心的调用或声明。
- **L2899 EN**: Closes the current lexical scope or compound statement.
  **L2899 CN**: 结束当前词法作用域或复合语句块。
- **L2900 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2900 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2901 EN**: Continues logic associated with callable symbol `insert`.
  **L2901 CN**: 继续与可调用符号 `insert` 相关的逻辑。
- **L2902 EN**: Executes a call or declaration centered on `std::make_pair`.
  **L2902 CN**: 执行以 `std::make_pair` 为核心的调用或声明。
- **L2903 EN**: Closes the current lexical scope or compound statement.
  **L2903 CN**: 结束当前词法作用域或复合语句块。
- **L2904 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2904 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 2905-2928

````cpp
      specificFuncs_.insert(std::make_pair(std::string{f.name}, &f));
    }
    for (const IntrinsicInterface &f : intrinsicSubroutine) {
      subroutines_.insert(std::make_pair(std::string{f.name}, &f));
    }
  }

  void SupplyBuiltins(const semantics::Scope &builtins) {
    builtinsScope_ = &builtins;
  }

  bool IsIntrinsic(const std::string &) const;
  bool IsIntrinsicFunction(const std::string &) const;
  bool IsIntrinsicSubroutine(const std::string &) const;
  bool IsDualIntrinsic(const std::string &) const;

  IntrinsicClass GetIntrinsicClass(const std::string &) const;
  std::string GetGenericIntrinsicName(const std::string &) const;

  std::optional<SpecificCall> Probe(
      const CallCharacteristics &, ActualArguments &, FoldingContext &) const;

  std::optional<SpecificIntrinsicFunctionInterface> IsSpecificIntrinsicFunction(
      const std::string &) const;
````
- **L2905 EN**: Executes a call or declaration centered on `specificFuncs_.insert`.
  **L2905 CN**: 执行以 `specificFuncs_.insert` 为核心的调用或声明。
- **L2906 EN**: Closes the current lexical scope or compound statement.
  **L2906 CN**: 结束当前词法作用域或复合语句块。
- **L2907 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2907 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2908 EN**: Executes a call or declaration centered on `subroutines_.insert`.
  **L2908 CN**: 执行以 `subroutines_.insert` 为核心的调用或声明。
- **L2909 EN**: Closes the current lexical scope or compound statement.
  **L2909 CN**: 结束当前词法作用域或复合语句块。
- **L2910 EN**: Closes the current lexical scope or compound statement.
  **L2910 CN**: 结束当前词法作用域或复合语句块。
- **L2911 EN**: Blank line separating nearby declarations or logic blocks.
  **L2911 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2912 EN**: Starts a function, method, lambda, or structured scope: `void SupplyBuiltins(const semantics::Scope &builtins) {`.
  **L2912 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void SupplyBuiltins(const semantics::Scope &builtins) {`。
- **L2913 EN**: Executes a standalone statement or declaration: `builtinsScope_ = &builtins;`.
  **L2913 CN**: 执行一条独立语句或声明：`builtinsScope_ = &builtins;`。
- **L2914 EN**: Closes the current lexical scope or compound statement.
  **L2914 CN**: 结束当前词法作用域或复合语句块。
- **L2915 EN**: Blank line separating nearby declarations or logic blocks.
  **L2915 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2916 EN**: Executes a call or declaration centered on `IsIntrinsic`.
  **L2916 CN**: 执行以 `IsIntrinsic` 为核心的调用或声明。
- **L2917 EN**: Executes a call or declaration centered on `IsIntrinsicFunction`.
  **L2917 CN**: 执行以 `IsIntrinsicFunction` 为核心的调用或声明。
- **L2918 EN**: Executes a call or declaration centered on `IsIntrinsicSubroutine`.
  **L2918 CN**: 执行以 `IsIntrinsicSubroutine` 为核心的调用或声明。
- **L2919 EN**: Executes a call or declaration centered on `IsDualIntrinsic`.
  **L2919 CN**: 执行以 `IsDualIntrinsic` 为核心的调用或声明。
- **L2920 EN**: Blank line separating nearby declarations or logic blocks.
  **L2920 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2921 EN**: Executes a call or declaration centered on `GetIntrinsicClass`.
  **L2921 CN**: 执行以 `GetIntrinsicClass` 为核心的调用或声明。
- **L2922 EN**: Executes a call or declaration centered on `GetGenericIntrinsicName`.
  **L2922 CN**: 执行以 `GetGenericIntrinsicName` 为核心的调用或声明。
- **L2923 EN**: Blank line separating nearby declarations or logic blocks.
  **L2923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2924 EN**: Continues logic associated with callable symbol `Probe`.
  **L2924 CN**: 继续与可调用符号 `Probe` 相关的逻辑。
- **L2925 EN**: Executes a standalone statement or declaration: `const CallCharacteristics &, ActualArguments &, FoldingContext &) const;`.
  **L2925 CN**: 执行一条独立语句或声明：`const CallCharacteristics &, ActualArguments &, FoldingContext &) const;`。
- **L2926 EN**: Blank line separating nearby declarations or logic blocks.
  **L2926 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2927 EN**: Continues logic associated with callable symbol `IsSpecificIntrinsicFunction`.
  **L2927 CN**: 继续与可调用符号 `IsSpecificIntrinsicFunction` 相关的逻辑。
- **L2928 EN**: Executes a standalone statement or declaration: `const std::string &) const;`.
  **L2928 CN**: 执行一条独立语句或声明：`const std::string &) const;`。

### Lines 2929-2952

````cpp

  llvm::raw_ostream &Dump(llvm::raw_ostream &) const;

private:
  DynamicType GetSpecificType(const TypePattern &) const;
  SpecificCall HandleNull(ActualArguments &, FoldingContext &) const;
  std::optional<SpecificCall> HandleC_F_Pointer(
      ActualArguments &, FoldingContext &) const;
  std::optional<SpecificCall> HandleC_F_Strpointer(
      ActualArguments &, FoldingContext &) const;
  std::optional<SpecificCall> HandleC_Loc(
      ActualArguments &, FoldingContext &) const;
  std::optional<SpecificCall> HandleC_Devloc(
      ActualArguments &, FoldingContext &) const;
  const std::string &ResolveAlias(const std::string &name) const {
    auto iter{aliases_.find(name)};
    return iter == aliases_.end() ? name : iter->second;
  }

  common::IntrinsicTypeDefaultKinds defaults_;
  std::multimap<std::string, const IntrinsicInterface *> genericFuncs_;
  std::multimap<std::string, const SpecificIntrinsicInterface *> specificFuncs_;
  std::multimap<std::string, const IntrinsicInterface *> subroutines_;
  const semantics::Scope *builtinsScope_{nullptr};
````
- **L2929 EN**: Blank line separating nearby declarations or logic blocks.
  **L2929 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2930 EN**: Executes a call or declaration centered on `&Dump`.
  **L2930 CN**: 执行以 `&Dump` 为核心的调用或声明。
- **L2931 EN**: Blank line separating nearby declarations or logic blocks.
  **L2931 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2932 EN**: Sets the following members to `private` access.
  **L2932 CN**: 将后续成员的访问级别设为 `private`。
- **L2933 EN**: Executes a call or declaration centered on `GetSpecificType`.
  **L2933 CN**: 执行以 `GetSpecificType` 为核心的调用或声明。
- **L2934 EN**: Executes a call or declaration centered on `HandleNull`.
  **L2934 CN**: 执行以 `HandleNull` 为核心的调用或声明。
- **L2935 EN**: Continues logic associated with callable symbol `HandleC_F_Pointer`.
  **L2935 CN**: 继续与可调用符号 `HandleC_F_Pointer` 相关的逻辑。
- **L2936 EN**: Executes a standalone statement or declaration: `ActualArguments &, FoldingContext &) const;`.
  **L2936 CN**: 执行一条独立语句或声明：`ActualArguments &, FoldingContext &) const;`。
- **L2937 EN**: Continues logic associated with callable symbol `HandleC_F_Strpointer`.
  **L2937 CN**: 继续与可调用符号 `HandleC_F_Strpointer` 相关的逻辑。
- **L2938 EN**: Executes a standalone statement or declaration: `ActualArguments &, FoldingContext &) const;`.
  **L2938 CN**: 执行一条独立语句或声明：`ActualArguments &, FoldingContext &) const;`。
- **L2939 EN**: Continues logic associated with callable symbol `HandleC_Loc`.
  **L2939 CN**: 继续与可调用符号 `HandleC_Loc` 相关的逻辑。
- **L2940 EN**: Executes a standalone statement or declaration: `ActualArguments &, FoldingContext &) const;`.
  **L2940 CN**: 执行一条独立语句或声明：`ActualArguments &, FoldingContext &) const;`。
- **L2941 EN**: Continues logic associated with callable symbol `HandleC_Devloc`.
  **L2941 CN**: 继续与可调用符号 `HandleC_Devloc` 相关的逻辑。
- **L2942 EN**: Executes a standalone statement or declaration: `ActualArguments &, FoldingContext &) const;`.
  **L2942 CN**: 执行一条独立语句或声明：`ActualArguments &, FoldingContext &) const;`。
- **L2943 EN**: Starts a function, method, lambda, or structured scope: `const std::string &ResolveAlias(const std::string &name) const {`.
  **L2943 CN**: 开始一个函数、方法、lambda 或结构化作用域：`const std::string &ResolveAlias(const std::string &name) const {`。
- **L2944 EN**: Executes a call or declaration centered on `iter{aliases_.find`.
  **L2944 CN**: 执行以 `iter{aliases_.find` 为核心的调用或声明。
- **L2945 EN**: Returns from the current function with `iter == aliases_.end() ? name : iter->second`.
  **L2945 CN**: 以 `iter == aliases_.end() ? name : iter->second` 从当前函数返回。
- **L2946 EN**: Closes the current lexical scope or compound statement.
  **L2946 CN**: 结束当前词法作用域或复合语句块。
- **L2947 EN**: Blank line separating nearby declarations or logic blocks.
  **L2947 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2948 EN**: Executes a standalone statement or declaration: `common::IntrinsicTypeDefaultKinds defaults_;`.
  **L2948 CN**: 执行一条独立语句或声明：`common::IntrinsicTypeDefaultKinds defaults_;`。
- **L2949 EN**: Executes a standalone statement or declaration: `std::multimap<std::string, const IntrinsicInterface *> genericFuncs_;`.
  **L2949 CN**: 执行一条独立语句或声明：`std::multimap<std::string, const IntrinsicInterface *> genericFuncs_;`。
- **L2950 EN**: Executes a standalone statement or declaration: `std::multimap<std::string, const SpecificIntrinsicInterface *> specificFuncs_;`.
  **L2950 CN**: 执行一条独立语句或声明：`std::multimap<std::string, const SpecificIntrinsicInterface *> specificFuncs_;`。
- **L2951 EN**: Executes a standalone statement or declaration: `std::multimap<std::string, const IntrinsicInterface *> subroutines_;`.
  **L2951 CN**: 执行一条独立语句或声明：`std::multimap<std::string, const IntrinsicInterface *> subroutines_;`。
- **L2952 EN**: Executes a standalone statement or declaration: `const semantics::Scope *builtinsScope_{nullptr};`.
  **L2952 CN**: 执行一条独立语句或声明：`const semantics::Scope *builtinsScope_{nullptr};`。

### Lines 2953-2976

````cpp
  std::map<std::string, std::string> aliases_;
  semantics::ParamValue assumedLen_{
      semantics::ParamValue::Assumed(common::TypeParamAttr::Len)};
};

bool IntrinsicProcTable::Implementation::IsIntrinsicFunction(
    const std::string &name0) const {
  const std::string &name{ResolveAlias(name0)};
  auto specificRange{specificFuncs_.equal_range(name)};
  if (specificRange.first != specificRange.second) {
    return true;
  }
  auto genericRange{genericFuncs_.equal_range(name)};
  if (genericRange.first != genericRange.second) {
    return true;
  }
  // special cases
  return name == "__builtin_c_loc" || name == "__builtin_c_devloc" ||
      name == "null";
}
bool IntrinsicProcTable::Implementation::IsIntrinsicSubroutine(
    const std::string &name0) const {
  const std::string &name{ResolveAlias(name0)};
  auto subrRange{subroutines_.equal_range(name)};
````
- **L2953 EN**: Executes a standalone statement or declaration: `std::map<std::string, std::string> aliases_;`.
  **L2953 CN**: 执行一条独立语句或声明：`std::map<std::string, std::string> aliases_;`。
- **L2954 EN**: Continues the surrounding expression or declaration: `semantics::ParamValue assumedLen_{`.
  **L2954 CN**: 继续构造周围的表达式或声明：`semantics::ParamValue assumedLen_{`。
- **L2955 EN**: Executes a call or declaration centered on `semantics::ParamValue::Assumed`.
  **L2955 CN**: 执行以 `semantics::ParamValue::Assumed` 为核心的调用或声明。
- **L2956 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2956 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2957 EN**: Blank line separating nearby declarations or logic blocks.
  **L2957 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2958 EN**: Continues logic associated with callable symbol `IsIntrinsicFunction`.
  **L2958 CN**: 继续与可调用符号 `IsIntrinsicFunction` 相关的逻辑。
- **L2959 EN**: Continues the surrounding expression or declaration: `const std::string &name0) const {`.
  **L2959 CN**: 继续构造周围的表达式或声明：`const std::string &name0) const {`。
- **L2960 EN**: Executes a call or declaration centered on `&name{ResolveAlias`.
  **L2960 CN**: 执行以 `&name{ResolveAlias` 为核心的调用或声明。
- **L2961 EN**: Executes a call or declaration centered on `specificRange{specificFuncs_.equal_range`.
  **L2961 CN**: 执行以 `specificRange{specificFuncs_.equal_range` 为核心的调用或声明。
- **L2962 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2962 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2963 EN**: Returns from the current function with `true`.
  **L2963 CN**: 以 `true` 从当前函数返回。
- **L2964 EN**: Closes the current lexical scope or compound statement.
  **L2964 CN**: 结束当前词法作用域或复合语句块。
- **L2965 EN**: Executes a call or declaration centered on `genericRange{genericFuncs_.equal_range`.
  **L2965 CN**: 执行以 `genericRange{genericFuncs_.equal_range` 为核心的调用或声明。
- **L2966 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2966 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2967 EN**: Returns from the current function with `true`.
  **L2967 CN**: 以 `true` 从当前函数返回。
- **L2968 EN**: Closes the current lexical scope or compound statement.
  **L2968 CN**: 结束当前词法作用域或复合语句块。
- **L2969 EN**: Comment explains nearby logic, intent, or metadata: `special cases`.
  **L2969 CN**: 注释说明附近代码的逻辑、意图或元数据：`special cases`。
- **L2970 EN**: Returns from the current function with `name == "__builtin_c_loc" || name == "__builtin_c_devloc" ||`.
  **L2970 CN**: 以 `name == "__builtin_c_loc" || name == "__builtin_c_devloc" ||` 从当前函数返回。
- **L2971 EN**: Executes a standalone statement or declaration: `name == "null";`.
  **L2971 CN**: 执行一条独立语句或声明：`name == "null";`。
- **L2972 EN**: Closes the current lexical scope or compound statement.
  **L2972 CN**: 结束当前词法作用域或复合语句块。
- **L2973 EN**: Continues logic associated with callable symbol `IsIntrinsicSubroutine`.
  **L2973 CN**: 继续与可调用符号 `IsIntrinsicSubroutine` 相关的逻辑。
- **L2974 EN**: Continues the surrounding expression or declaration: `const std::string &name0) const {`.
  **L2974 CN**: 继续构造周围的表达式或声明：`const std::string &name0) const {`。
- **L2975 EN**: Executes a call or declaration centered on `&name{ResolveAlias`.
  **L2975 CN**: 执行以 `&name{ResolveAlias` 为核心的调用或声明。
- **L2976 EN**: Executes a call or declaration centered on `subrRange{subroutines_.equal_range`.
  **L2976 CN**: 执行以 `subrRange{subroutines_.equal_range` 为核心的调用或声明。

### Lines 2977-3000

````cpp
  if (subrRange.first != subrRange.second) {
    return true;
  }
  // special cases
  return name == "__builtin_c_f_pointer" || name == "__builtin_c_f_strpointer";
}
bool IntrinsicProcTable::Implementation::IsIntrinsic(
    const std::string &name) const {
  return IsIntrinsicFunction(name) || IsIntrinsicSubroutine(name);
}
bool IntrinsicProcTable::Implementation::IsDualIntrinsic(
    const std::string &name) const {
  // Collection for some intrinsics with function and subroutine form,
  // in order to pass the semantic check.
  static const std::string dualIntrinsic[]{{"chdir"}, {"etime"}, {"fseek"},
      {"ftell"}, {"getcwd"}, {"hostnm"}, {"putenv"}, {"rename"}, {"second"},
      {"system"}, {"unlink"}};
  return llvm::is_contained(dualIntrinsic, name);
}

IntrinsicClass IntrinsicProcTable::Implementation::GetIntrinsicClass(
    const std::string &name) const {
  auto specificIntrinsic{specificFuncs_.find(name)};
  if (specificIntrinsic != specificFuncs_.end()) {
````
- **L2977 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2977 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2978 EN**: Returns from the current function with `true`.
  **L2978 CN**: 以 `true` 从当前函数返回。
- **L2979 EN**: Closes the current lexical scope or compound statement.
  **L2979 CN**: 结束当前词法作用域或复合语句块。
- **L2980 EN**: Comment explains nearby logic, intent, or metadata: `special cases`.
  **L2980 CN**: 注释说明附近代码的逻辑、意图或元数据：`special cases`。
- **L2981 EN**: Returns from the current function with `name == "__builtin_c_f_pointer" || name == "__builtin_c_f_strpointer"`.
  **L2981 CN**: 以 `name == "__builtin_c_f_pointer" || name == "__builtin_c_f_strpointer"` 从当前函数返回。
- **L2982 EN**: Closes the current lexical scope or compound statement.
  **L2982 CN**: 结束当前词法作用域或复合语句块。
- **L2983 EN**: Continues logic associated with callable symbol `IsIntrinsic`.
  **L2983 CN**: 继续与可调用符号 `IsIntrinsic` 相关的逻辑。
- **L2984 EN**: Continues the surrounding expression or declaration: `const std::string &name) const {`.
  **L2984 CN**: 继续构造周围的表达式或声明：`const std::string &name) const {`。
- **L2985 EN**: Returns from the current function with `IsIntrinsicFunction(name) || IsIntrinsicSubroutine(name)`.
  **L2985 CN**: 以 `IsIntrinsicFunction(name) || IsIntrinsicSubroutine(name)` 从当前函数返回。
- **L2986 EN**: Closes the current lexical scope or compound statement.
  **L2986 CN**: 结束当前词法作用域或复合语句块。
- **L2987 EN**: Continues logic associated with callable symbol `IsDualIntrinsic`.
  **L2987 CN**: 继续与可调用符号 `IsDualIntrinsic` 相关的逻辑。
- **L2988 EN**: Continues the surrounding expression or declaration: `const std::string &name) const {`.
  **L2988 CN**: 继续构造周围的表达式或声明：`const std::string &name) const {`。
- **L2989 EN**: Comment explains nearby logic, intent, or metadata: `Collection for some intrinsics with function and subroutine form,`.
  **L2989 CN**: 注释说明附近代码的逻辑、意图或元数据：`Collection for some intrinsics with function and subroutine form,`。
- **L2990 EN**: Comment explains nearby logic, intent, or metadata: `in order to pass the semantic check.`.
  **L2990 CN**: 注释说明附近代码的逻辑、意图或元数据：`in order to pass the semantic check.`。
- **L2991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static const std::string dualIntrinsic[]{{"chdir"}, {"etime"}, {"fseek"},`.
  **L2991 CN**: 继续一个多行参数列表、初始化器或聚合项：`static const std::string dualIntrinsic[]{{"chdir"}, {"etime"}, {"fseek"},`。
- **L2992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ftell"}, {"getcwd"}, {"hostnm"}, {"putenv"}, {"rename"}, {"second"},`.
  **L2992 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ftell"}, {"getcwd"}, {"hostnm"}, {"putenv"}, {"rename"}, {"second"},`。
- **L2993 EN**: Executes a standalone statement or declaration: `{"system"}, {"unlink"}};`.
  **L2993 CN**: 执行一条独立语句或声明：`{"system"}, {"unlink"}};`。
- **L2994 EN**: Returns from the current function with `llvm::is_contained(dualIntrinsic, name)`.
  **L2994 CN**: 以 `llvm::is_contained(dualIntrinsic, name)` 从当前函数返回。
- **L2995 EN**: Closes the current lexical scope or compound statement.
  **L2995 CN**: 结束当前词法作用域或复合语句块。
- **L2996 EN**: Blank line separating nearby declarations or logic blocks.
  **L2996 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2997 EN**: Continues logic associated with callable symbol `GetIntrinsicClass`.
  **L2997 CN**: 继续与可调用符号 `GetIntrinsicClass` 相关的逻辑。
- **L2998 EN**: Continues the surrounding expression or declaration: `const std::string &name) const {`.
  **L2998 CN**: 继续构造周围的表达式或声明：`const std::string &name) const {`。
- **L2999 EN**: Executes a call or declaration centered on `specificIntrinsic{specificFuncs_.find`.
  **L2999 CN**: 执行以 `specificIntrinsic{specificFuncs_.find` 为核心的调用或声明。
- **L3000 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3000 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3001-3024

````cpp
    return specificIntrinsic->second->intrinsicClass;
  }
  auto genericIntrinsic{genericFuncs_.find(name)};
  if (genericIntrinsic != genericFuncs_.end()) {
    return genericIntrinsic->second->intrinsicClass;
  }
  auto subrIntrinsic{subroutines_.find(name)};
  if (subrIntrinsic != subroutines_.end()) {
    return subrIntrinsic->second->intrinsicClass;
  }
  return IntrinsicClass::noClass;
}

std::string IntrinsicProcTable::Implementation::GetGenericIntrinsicName(
    const std::string &name) const {
  auto specificIntrinsic{specificFuncs_.find(name)};
  if (specificIntrinsic != specificFuncs_.end()) {
    if (const char *genericName{specificIntrinsic->second->generic}) {
      return {genericName};
    }
  }
  return name;
}

````
- **L3001 EN**: Returns from the current function with `specificIntrinsic->second->intrinsicClass`.
  **L3001 CN**: 以 `specificIntrinsic->second->intrinsicClass` 从当前函数返回。
- **L3002 EN**: Closes the current lexical scope or compound statement.
  **L3002 CN**: 结束当前词法作用域或复合语句块。
- **L3003 EN**: Executes a call or declaration centered on `genericIntrinsic{genericFuncs_.find`.
  **L3003 CN**: 执行以 `genericIntrinsic{genericFuncs_.find` 为核心的调用或声明。
- **L3004 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3004 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3005 EN**: Returns from the current function with `genericIntrinsic->second->intrinsicClass`.
  **L3005 CN**: 以 `genericIntrinsic->second->intrinsicClass` 从当前函数返回。
- **L3006 EN**: Closes the current lexical scope or compound statement.
  **L3006 CN**: 结束当前词法作用域或复合语句块。
- **L3007 EN**: Executes a call or declaration centered on `subrIntrinsic{subroutines_.find`.
  **L3007 CN**: 执行以 `subrIntrinsic{subroutines_.find` 为核心的调用或声明。
- **L3008 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3008 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3009 EN**: Returns from the current function with `subrIntrinsic->second->intrinsicClass`.
  **L3009 CN**: 以 `subrIntrinsic->second->intrinsicClass` 从当前函数返回。
- **L3010 EN**: Closes the current lexical scope or compound statement.
  **L3010 CN**: 结束当前词法作用域或复合语句块。
- **L3011 EN**: Returns from the current function with `IntrinsicClass::noClass`.
  **L3011 CN**: 以 `IntrinsicClass::noClass` 从当前函数返回。
- **L3012 EN**: Closes the current lexical scope or compound statement.
  **L3012 CN**: 结束当前词法作用域或复合语句块。
- **L3013 EN**: Blank line separating nearby declarations or logic blocks.
  **L3013 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3014 EN**: Continues logic associated with callable symbol `GetGenericIntrinsicName`.
  **L3014 CN**: 继续与可调用符号 `GetGenericIntrinsicName` 相关的逻辑。
- **L3015 EN**: Continues the surrounding expression or declaration: `const std::string &name) const {`.
  **L3015 CN**: 继续构造周围的表达式或声明：`const std::string &name) const {`。
- **L3016 EN**: Executes a call or declaration centered on `specificIntrinsic{specificFuncs_.find`.
  **L3016 CN**: 执行以 `specificIntrinsic{specificFuncs_.find` 为核心的调用或声明。
- **L3017 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3017 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3018 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3018 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3019 EN**: Returns from the current function with `{genericName}`.
  **L3019 CN**: 以 `{genericName}` 从当前函数返回。
- **L3020 EN**: Closes the current lexical scope or compound statement.
  **L3020 CN**: 结束当前词法作用域或复合语句块。
- **L3021 EN**: Closes the current lexical scope or compound statement.
  **L3021 CN**: 结束当前词法作用域或复合语句块。
- **L3022 EN**: Returns from the current function with `name`.
  **L3022 CN**: 以 `name` 从当前函数返回。
- **L3023 EN**: Closes the current lexical scope or compound statement.
  **L3023 CN**: 结束当前词法作用域或复合语句块。
- **L3024 EN**: Blank line separating nearby declarations or logic blocks.
  **L3024 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3025-3048

````cpp
bool CheckAndRearrangeArguments(ActualArguments &arguments,
    parser::ContextualMessages &messages, const char *const dummyKeywords[],
    std::size_t trailingOptionals) {
  std::size_t numDummies{0};
  while (dummyKeywords[numDummies]) {
    ++numDummies;
  }
  CHECK(trailingOptionals <= numDummies);
  if (arguments.size() > numDummies) {
    messages.Say("Too many actual arguments (%zd > %zd)"_err_en_US,
        arguments.size(), numDummies);
    return false;
  }
  ActualArguments rearranged(numDummies);
  bool anyKeywords{false};
  std::size_t position{0};
  for (std::optional<ActualArgument> &arg : arguments) {
    std::size_t dummyIndex{0};
    if (arg && arg->keyword()) {
      anyKeywords = true;
      for (; dummyIndex < numDummies; ++dummyIndex) {
        if (*arg->keyword() == dummyKeywords[dummyIndex]) {
          break;
        }
````
- **L3025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool CheckAndRearrangeArguments(ActualArguments &arguments,`.
  **L3025 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool CheckAndRearrangeArguments(ActualArguments &arguments,`。
- **L3026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ContextualMessages &messages, const char *const dummyKeywords[],`.
  **L3026 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ContextualMessages &messages, const char *const dummyKeywords[],`。
- **L3027 EN**: Continues the surrounding expression or declaration: `std::size_t trailingOptionals) {`.
  **L3027 CN**: 继续构造周围的表达式或声明：`std::size_t trailingOptionals) {`。
- **L3028 EN**: Executes a standalone statement or declaration: `std::size_t numDummies{0};`.
  **L3028 CN**: 执行一条独立语句或声明：`std::size_t numDummies{0};`。
- **L3029 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L3029 CN**: 开始 `while` 控制流语句并计算其条件。
- **L3030 EN**: Executes a standalone statement or declaration: `++numDummies;`.
  **L3030 CN**: 执行一条独立语句或声明：`++numDummies;`。
- **L3031 EN**: Closes the current lexical scope or compound statement.
  **L3031 CN**: 结束当前词法作用域或复合语句块。
- **L3032 EN**: Executes a call or declaration centered on `CHECK`.
  **L3032 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L3033 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3033 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say("Too many actual arguments (%zd > %zd)"_err_en_US,`.
  **L3034 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say("Too many actual arguments (%zd > %zd)"_err_en_US,`。
- **L3035 EN**: Executes a call or declaration centered on `arguments.size`.
  **L3035 CN**: 执行以 `arguments.size` 为核心的调用或声明。
- **L3036 EN**: Returns from the current function with `false`.
  **L3036 CN**: 以 `false` 从当前函数返回。
- **L3037 EN**: Closes the current lexical scope or compound statement.
  **L3037 CN**: 结束当前词法作用域或复合语句块。
- **L3038 EN**: Executes a call or declaration centered on `rearranged`.
  **L3038 CN**: 执行以 `rearranged` 为核心的调用或声明。
- **L3039 EN**: Executes a standalone statement or declaration: `bool anyKeywords{false};`.
  **L3039 CN**: 执行一条独立语句或声明：`bool anyKeywords{false};`。
- **L3040 EN**: Executes a standalone statement or declaration: `std::size_t position{0};`.
  **L3040 CN**: 执行一条独立语句或声明：`std::size_t position{0};`。
- **L3041 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3041 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3042 EN**: Executes a standalone statement or declaration: `std::size_t dummyIndex{0};`.
  **L3042 CN**: 执行一条独立语句或声明：`std::size_t dummyIndex{0};`。
- **L3043 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3043 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3044 EN**: Executes a standalone statement or declaration: `anyKeywords = true;`.
  **L3044 CN**: 执行一条独立语句或声明：`anyKeywords = true;`。
- **L3045 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3045 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3046 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3046 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3047 EN**: Exits the nearest loop or switch statement.
  **L3047 CN**: 退出最近的循环或 switch 语句。
- **L3048 EN**: Closes the current lexical scope or compound statement.
  **L3048 CN**: 结束当前词法作用域或复合语句块。

### Lines 3049-3072

````cpp
      }
      if (dummyIndex >= numDummies) {
        messages.Say(*arg->keyword(),
            "Unknown argument keyword '%s='"_err_en_US, *arg->keyword());
        return false;
      }
    } else if (anyKeywords) {
      messages.Say(arg ? arg->sourceLocation() : messages.at(),
          "A positional actual argument may not appear after any keyword arguments"_err_en_US);
      return false;
    } else {
      dummyIndex = position++;
    }
    if (rearranged[dummyIndex]) {
      messages.Say(arg ? arg->sourceLocation() : messages.at(),
          "Dummy argument '%s=' appears more than once"_err_en_US,
          dummyKeywords[dummyIndex]);
      return false;
    }
    rearranged[dummyIndex] = std::move(arg);
    arg.reset();
  }
  bool anyMissing{false};
  for (std::size_t j{0}; j < numDummies - trailingOptionals; ++j) {
````
- **L3049 EN**: Closes the current lexical scope or compound statement.
  **L3049 CN**: 结束当前词法作用域或复合语句块。
- **L3050 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3050 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*arg->keyword(),`.
  **L3051 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*arg->keyword(),`。
- **L3052 EN**: Executes a call or declaration centered on `*arg->keyword`.
  **L3052 CN**: 执行以 `*arg->keyword` 为核心的调用或声明。
- **L3053 EN**: Returns from the current function with `false`.
  **L3053 CN**: 以 `false` 从当前函数返回。
- **L3054 EN**: Closes the current lexical scope or compound statement.
  **L3054 CN**: 结束当前词法作用域或复合语句块。
- **L3055 EN**: Transitions from the previous branch into an `else if` condition.
  **L3055 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg ? arg->sourceLocation() : messages.at(),`.
  **L3056 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg ? arg->sourceLocation() : messages.at(),`。
- **L3057 EN**: Executes a standalone statement or declaration: `"A positional actual argument may not appear after any keyword arguments"_err_en_US);`.
  **L3057 CN**: 执行一条独立语句或声明：`"A positional actual argument may not appear after any keyword arguments"_err_en_US);`。
- **L3058 EN**: Returns from the current function with `false`.
  **L3058 CN**: 以 `false` 从当前函数返回。
- **L3059 EN**: Transitions from the previous branch into the alternative path.
  **L3059 CN**: 从前一个分支过渡到备选路径。
- **L3060 EN**: Executes a standalone statement or declaration: `dummyIndex = position++;`.
  **L3060 CN**: 执行一条独立语句或声明：`dummyIndex = position++;`。
- **L3061 EN**: Closes the current lexical scope or compound statement.
  **L3061 CN**: 结束当前词法作用域或复合语句块。
- **L3062 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3062 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg ? arg->sourceLocation() : messages.at(),`.
  **L3063 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg ? arg->sourceLocation() : messages.at(),`。
- **L3064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s=' appears more than once"_err_en_US,`.
  **L3064 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s=' appears more than once"_err_en_US,`。
- **L3065 EN**: Executes a standalone statement or declaration: `dummyKeywords[dummyIndex]);`.
  **L3065 CN**: 执行一条独立语句或声明：`dummyKeywords[dummyIndex]);`。
- **L3066 EN**: Returns from the current function with `false`.
  **L3066 CN**: 以 `false` 从当前函数返回。
- **L3067 EN**: Closes the current lexical scope or compound statement.
  **L3067 CN**: 结束当前词法作用域或复合语句块。
- **L3068 EN**: Executes a call or declaration centered on `std::move`.
  **L3068 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3069 EN**: Executes a call or declaration centered on `arg.reset`.
  **L3069 CN**: 执行以 `arg.reset` 为核心的调用或声明。
- **L3070 EN**: Closes the current lexical scope or compound statement.
  **L3070 CN**: 结束当前词法作用域或复合语句块。
- **L3071 EN**: Executes a standalone statement or declaration: `bool anyMissing{false};`.
  **L3071 CN**: 执行一条独立语句或声明：`bool anyMissing{false};`。
- **L3072 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3072 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 3073-3096

````cpp
    if (!rearranged[j]) {
      messages.Say("Dummy argument '%s=' is absent and not OPTIONAL"_err_en_US,
          dummyKeywords[j]);
      anyMissing = true;
    }
  }
  arguments = std::move(rearranged);
  return !anyMissing;
}

// The NULL() intrinsic is a special case.
SpecificCall IntrinsicProcTable::Implementation::HandleNull(
    ActualArguments &arguments, FoldingContext &context) const {
  static const char *const keywords[]{"mold", nullptr};
  if (CheckAndRearrangeArguments(arguments, context.messages(), keywords, 1) &&
      arguments[0]) {
    Expr<SomeType> *mold{arguments[0]->UnwrapExpr()};
    bool isBareNull{IsBareNullPointer(mold)};
    if (isBareNull) {
      // NULL(NULL()), NULL(NULL(NULL())), &c. are all just NULL()
      mold = nullptr;
    }
    if (mold) {
      if (semantics::IsAssumedRank(*arguments[0])) {
````
- **L3073 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3073 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say("Dummy argument '%s=' is absent and not OPTIONAL"_err_en_US,`.
  **L3074 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say("Dummy argument '%s=' is absent and not OPTIONAL"_err_en_US,`。
- **L3075 EN**: Executes a standalone statement or declaration: `dummyKeywords[j]);`.
  **L3075 CN**: 执行一条独立语句或声明：`dummyKeywords[j]);`。
- **L3076 EN**: Executes a standalone statement or declaration: `anyMissing = true;`.
  **L3076 CN**: 执行一条独立语句或声明：`anyMissing = true;`。
- **L3077 EN**: Closes the current lexical scope or compound statement.
  **L3077 CN**: 结束当前词法作用域或复合语句块。
- **L3078 EN**: Closes the current lexical scope or compound statement.
  **L3078 CN**: 结束当前词法作用域或复合语句块。
- **L3079 EN**: Executes a call or declaration centered on `std::move`.
  **L3079 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3080 EN**: Returns from the current function with `!anyMissing`.
  **L3080 CN**: 以 `!anyMissing` 从当前函数返回。
- **L3081 EN**: Closes the current lexical scope or compound statement.
  **L3081 CN**: 结束当前词法作用域或复合语句块。
- **L3082 EN**: Blank line separating nearby declarations or logic blocks.
  **L3082 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3083 EN**: Comment explains nearby logic, intent, or metadata: `The NULL() intrinsic is a special case.`.
  **L3083 CN**: 注释说明附近代码的逻辑、意图或元数据：`The NULL() intrinsic is a special case.`。
- **L3084 EN**: Continues logic associated with callable symbol `HandleNull`.
  **L3084 CN**: 继续与可调用符号 `HandleNull` 相关的逻辑。
- **L3085 EN**: Continues the surrounding expression or declaration: `ActualArguments &arguments, FoldingContext &context) const {`.
  **L3085 CN**: 继续构造周围的表达式或声明：`ActualArguments &arguments, FoldingContext &context) const {`。
- **L3086 EN**: Executes a standalone statement or declaration: `static const char *const keywords[]{"mold", nullptr};`.
  **L3086 CN**: 执行一条独立语句或声明：`static const char *const keywords[]{"mold", nullptr};`。
- **L3087 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3087 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3088 EN**: Continues the surrounding expression or declaration: `arguments[0]) {`.
  **L3088 CN**: 继续构造周围的表达式或声明：`arguments[0]) {`。
- **L3089 EN**: Executes a call or declaration centered on `*mold{arguments[0]->UnwrapExpr`.
  **L3089 CN**: 执行以 `*mold{arguments[0]->UnwrapExpr` 为核心的调用或声明。
- **L3090 EN**: Executes a call or declaration centered on `isBareNull{IsBareNullPointer`.
  **L3090 CN**: 执行以 `isBareNull{IsBareNullPointer` 为核心的调用或声明。
- **L3091 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3091 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3092 EN**: Comment explains nearby logic, intent, or metadata: `NULL(NULL()), NULL(NULL(NULL())), &c. are all just NULL()`.
  **L3092 CN**: 注释说明附近代码的逻辑、意图或元数据：`NULL(NULL()), NULL(NULL(NULL())), &c. are all just NULL()`。
- **L3093 EN**: Executes a standalone statement or declaration: `mold = nullptr;`.
  **L3093 CN**: 执行一条独立语句或声明：`mold = nullptr;`。
- **L3094 EN**: Closes the current lexical scope or compound statement.
  **L3094 CN**: 结束当前词法作用域或复合语句块。
- **L3095 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3095 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3096 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3096 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3097-3120

````cpp
        context.messages().Say(arguments[0]->sourceLocation(),
            "MOLD= argument to NULL() must not be assumed-rank"_err_en_US);
      }
      bool isProcPtrTarget{
          IsProcedurePointerTarget(*mold) && !IsNullObjectPointer(mold)};
      if (isProcPtrTarget || IsAllocatableOrPointerObject(*mold)) {
        characteristics::DummyArguments args;
        std::optional<characteristics::FunctionResult> fResult;
        bool isAllocatableMold{false};
        if (isProcPtrTarget) {
          // MOLD= procedure pointer
          std::optional<characteristics::Procedure> procPointer;
          if (IsNullProcedurePointer(mold)) {
            procPointer =
                characteristics::Procedure::Characterize(*mold, context);
          } else {
            const Symbol *last{GetLastSymbol(*mold)};
            procPointer =
                characteristics::Procedure::Characterize(DEREF(last), context);
          }
          // procPointer is vacant if there was an error with the analysis
          // associated with the procedure pointer
          if (procPointer) {
            args.emplace_back("mold"s,
````
- **L3097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3097 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3098 EN**: Executes a call or declaration centered on `NULL`.
  **L3098 CN**: 执行以 `NULL` 为核心的调用或声明。
- **L3099 EN**: Closes the current lexical scope or compound statement.
  **L3099 CN**: 结束当前词法作用域或复合语句块。
- **L3100 EN**: Continues the surrounding expression or declaration: `bool isProcPtrTarget{`.
  **L3100 CN**: 继续构造周围的表达式或声明：`bool isProcPtrTarget{`。
- **L3101 EN**: Executes a call or declaration centered on `IsProcedurePointerTarget`.
  **L3101 CN**: 执行以 `IsProcedurePointerTarget` 为核心的调用或声明。
- **L3102 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3102 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3103 EN**: Executes a standalone statement or declaration: `characteristics::DummyArguments args;`.
  **L3103 CN**: 执行一条独立语句或声明：`characteristics::DummyArguments args;`。
- **L3104 EN**: Executes a standalone statement or declaration: `std::optional<characteristics::FunctionResult> fResult;`.
  **L3104 CN**: 执行一条独立语句或声明：`std::optional<characteristics::FunctionResult> fResult;`。
- **L3105 EN**: Executes a standalone statement or declaration: `bool isAllocatableMold{false};`.
  **L3105 CN**: 执行一条独立语句或声明：`bool isAllocatableMold{false};`。
- **L3106 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3106 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3107 EN**: Comment explains nearby logic, intent, or metadata: `MOLD= procedure pointer`.
  **L3107 CN**: 注释说明附近代码的逻辑、意图或元数据：`MOLD= procedure pointer`。
- **L3108 EN**: Executes a standalone statement or declaration: `std::optional<characteristics::Procedure> procPointer;`.
  **L3108 CN**: 执行一条独立语句或声明：`std::optional<characteristics::Procedure> procPointer;`。
- **L3109 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3109 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3110 EN**: Continues the surrounding expression or declaration: `procPointer =`.
  **L3110 CN**: 继续构造周围的表达式或声明：`procPointer =`。
- **L3111 EN**: Executes a call or declaration centered on `characteristics::Procedure::Characterize`.
  **L3111 CN**: 执行以 `characteristics::Procedure::Characterize` 为核心的调用或声明。
- **L3112 EN**: Transitions from the previous branch into the alternative path.
  **L3112 CN**: 从前一个分支过渡到备选路径。
- **L3113 EN**: Executes a call or declaration centered on `*last{GetLastSymbol`.
  **L3113 CN**: 执行以 `*last{GetLastSymbol` 为核心的调用或声明。
- **L3114 EN**: Continues the surrounding expression or declaration: `procPointer =`.
  **L3114 CN**: 继续构造周围的表达式或声明：`procPointer =`。
- **L3115 EN**: Executes a call or declaration centered on `characteristics::Procedure::Characterize`.
  **L3115 CN**: 执行以 `characteristics::Procedure::Characterize` 为核心的调用或声明。
- **L3116 EN**: Closes the current lexical scope or compound statement.
  **L3116 CN**: 结束当前词法作用域或复合语句块。
- **L3117 EN**: Comment explains nearby logic, intent, or metadata: `procPointer is vacant if there was an error with the analysis`.
  **L3117 CN**: 注释说明附近代码的逻辑、意图或元数据：`procPointer is vacant if there was an error with the analysis`。
- **L3118 EN**: Comment explains nearby logic, intent, or metadata: `associated with the procedure pointer`.
  **L3118 CN**: 注释说明附近代码的逻辑、意图或元数据：`associated with the procedure pointer`。
- **L3119 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3119 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `args.emplace_back("mold"s,`.
  **L3120 CN**: 继续一个多行参数列表、初始化器或聚合项：`args.emplace_back("mold"s,`。

### Lines 3121-3144

````cpp
                characteristics::DummyProcedure{common::Clone(*procPointer)});
            fResult.emplace(std::move(*procPointer));
          }
        } else if (auto type{mold->GetType()}) {
          // MOLD= object pointer or allocatable
          characteristics::TypeAndShape typeAndShape{
              *type, GetShape(context, *mold)};
          args.emplace_back(
              "mold"s, characteristics::DummyDataObject{typeAndShape});
          fResult.emplace(std::move(typeAndShape));
          isAllocatableMold = IsAllocatableDesignator(*mold);
        } else {
          context.messages().Say(arguments[0]->sourceLocation(),
              "MOLD= argument to NULL() lacks type"_err_en_US);
        }
        if (fResult) {
          fResult->attrs.set(characteristics::FunctionResult::Attr::Pointer);
          characteristics::Procedure::Attrs attrs;
          attrs.set(isAllocatableMold
                  ? characteristics::Procedure::Attr::NullAllocatable
                  : characteristics::Procedure::Attr::NullPointer);
          characteristics::Procedure chars{
              std::move(*fResult), std::move(args), attrs};
          return SpecificCall{SpecificIntrinsic{"null"s, std::move(chars)},
````
- **L3121 EN**: Executes a call or declaration centered on `characteristics::DummyProcedure{common::Clone`.
  **L3121 CN**: 执行以 `characteristics::DummyProcedure{common::Clone` 为核心的调用或声明。
- **L3122 EN**: Executes a call or declaration centered on `fResult.emplace`.
  **L3122 CN**: 执行以 `fResult.emplace` 为核心的调用或声明。
- **L3123 EN**: Closes the current lexical scope or compound statement.
  **L3123 CN**: 结束当前词法作用域或复合语句块。
- **L3124 EN**: Transitions from the previous branch into an `else if` condition.
  **L3124 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3125 EN**: Comment explains nearby logic, intent, or metadata: `MOLD= object pointer or allocatable`.
  **L3125 CN**: 注释说明附近代码的逻辑、意图或元数据：`MOLD= object pointer or allocatable`。
- **L3126 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape typeAndShape{`.
  **L3126 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape typeAndShape{`。
- **L3127 EN**: Comment explains nearby logic, intent, or metadata: `type, GetShape(context, *mold)};`.
  **L3127 CN**: 注释说明附近代码的逻辑、意图或元数据：`type, GetShape(context, *mold)};`。
- **L3128 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L3128 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L3129 EN**: Executes a standalone statement or declaration: `"mold"s, characteristics::DummyDataObject{typeAndShape});`.
  **L3129 CN**: 执行一条独立语句或声明：`"mold"s, characteristics::DummyDataObject{typeAndShape});`。
- **L3130 EN**: Executes a call or declaration centered on `fResult.emplace`.
  **L3130 CN**: 执行以 `fResult.emplace` 为核心的调用或声明。
- **L3131 EN**: Executes a call or declaration centered on `IsAllocatableDesignator`.
  **L3131 CN**: 执行以 `IsAllocatableDesignator` 为核心的调用或声明。
- **L3132 EN**: Transitions from the previous branch into the alternative path.
  **L3132 CN**: 从前一个分支过渡到备选路径。
- **L3133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3133 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3134 EN**: Executes a call or declaration centered on `NULL`.
  **L3134 CN**: 执行以 `NULL` 为核心的调用或声明。
- **L3135 EN**: Closes the current lexical scope or compound statement.
  **L3135 CN**: 结束当前词法作用域或复合语句块。
- **L3136 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3136 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3137 EN**: Executes a call or declaration centered on `fResult->attrs.set`.
  **L3137 CN**: 执行以 `fResult->attrs.set` 为核心的调用或声明。
- **L3138 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attrs attrs;`.
  **L3138 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attrs attrs;`。
- **L3139 EN**: Continues logic associated with callable symbol `set`.
  **L3139 CN**: 继续与可调用符号 `set` 相关的逻辑。
- **L3140 EN**: Continues the surrounding expression or declaration: `? characteristics::Procedure::Attr::NullAllocatable`.
  **L3140 CN**: 继续构造周围的表达式或声明：`? characteristics::Procedure::Attr::NullAllocatable`。
- **L3141 EN**: Executes a standalone statement or declaration: `: characteristics::Procedure::Attr::NullPointer);`.
  **L3141 CN**: 执行一条独立语句或声明：`: characteristics::Procedure::Attr::NullPointer);`。
- **L3142 EN**: Continues the surrounding expression or declaration: `characteristics::Procedure chars{`.
  **L3142 CN**: 继续构造周围的表达式或声明：`characteristics::Procedure chars{`。
- **L3143 EN**: Executes a call or declaration centered on `std::move`.
  **L3143 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3144 EN**: Returns from the current function with `SpecificCall{SpecificIntrinsic{"null"s, std::move(chars)},`.
  **L3144 CN**: 以 `SpecificCall{SpecificIntrinsic{"null"s, std::move(chars)},` 从当前函数返回。

### Lines 3145-3168

````cpp
              std::move(arguments)};
        }
      }
    }
    if (!isBareNull) {
      context.messages().Say(arguments[0]->sourceLocation(),
          "MOLD= argument to NULL() must be a pointer or allocatable"_err_en_US);
    }
  }
  characteristics::Procedure::Attrs attrs;
  attrs.set(characteristics::Procedure::Attr::NullPointer);
  attrs.set(characteristics::Procedure::Attr::Pure);
  arguments.clear();
  return SpecificCall{
      SpecificIntrinsic{"null"s,
          characteristics::Procedure{characteristics::DummyArguments{}, attrs}},
      std::move(arguments)};
}

// Subroutine C_F_POINTER(CPTR=,FPTR=[,SHAPE=]) from
// intrinsic module ISO_C_BINDING (18.2.3.3)
std::optional<SpecificCall>
IntrinsicProcTable::Implementation::HandleC_F_Pointer(
    ActualArguments &arguments, FoldingContext &context) const {
````
- **L3145 EN**: Executes a call or declaration centered on `std::move`.
  **L3145 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3146 EN**: Closes the current lexical scope or compound statement.
  **L3146 CN**: 结束当前词法作用域或复合语句块。
- **L3147 EN**: Closes the current lexical scope or compound statement.
  **L3147 CN**: 结束当前词法作用域或复合语句块。
- **L3148 EN**: Closes the current lexical scope or compound statement.
  **L3148 CN**: 结束当前词法作用域或复合语句块。
- **L3149 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3149 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3150 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3151 EN**: Executes a call or declaration centered on `NULL`.
  **L3151 CN**: 执行以 `NULL` 为核心的调用或声明。
- **L3152 EN**: Closes the current lexical scope or compound statement.
  **L3152 CN**: 结束当前词法作用域或复合语句块。
- **L3153 EN**: Closes the current lexical scope or compound statement.
  **L3153 CN**: 结束当前词法作用域或复合语句块。
- **L3154 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attrs attrs;`.
  **L3154 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attrs attrs;`。
- **L3155 EN**: Executes a call or declaration centered on `attrs.set`.
  **L3155 CN**: 执行以 `attrs.set` 为核心的调用或声明。
- **L3156 EN**: Executes a call or declaration centered on `attrs.set`.
  **L3156 CN**: 执行以 `attrs.set` 为核心的调用或声明。
- **L3157 EN**: Executes a call or declaration centered on `arguments.clear`.
  **L3157 CN**: 执行以 `arguments.clear` 为核心的调用或声明。
- **L3158 EN**: Returns from the current function with `SpecificCall{`.
  **L3158 CN**: 以 `SpecificCall{` 从当前函数返回。
- **L3159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SpecificIntrinsic{"null"s,`.
  **L3159 CN**: 继续一个多行参数列表、初始化器或聚合项：`SpecificIntrinsic{"null"s,`。
- **L3160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::Procedure{characteristics::DummyArguments{}, attrs}},`.
  **L3160 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::Procedure{characteristics::DummyArguments{}, attrs}},`。
- **L3161 EN**: Executes a call or declaration centered on `std::move`.
  **L3161 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3162 EN**: Closes the current lexical scope or compound statement.
  **L3162 CN**: 结束当前词法作用域或复合语句块。
- **L3163 EN**: Blank line separating nearby declarations or logic blocks.
  **L3163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3164 EN**: Comment explains nearby logic, intent, or metadata: `Subroutine C_F_POINTER(CPTR=,FPTR=[,SHAPE=]) from`.
  **L3164 CN**: 注释说明附近代码的逻辑、意图或元数据：`Subroutine C_F_POINTER(CPTR=,FPTR=[,SHAPE=]) from`。
- **L3165 EN**: Comment explains nearby logic, intent, or metadata: `intrinsic module ISO_C_BINDING (18.2.3.3)`.
  **L3165 CN**: 注释说明附近代码的逻辑、意图或元数据：`intrinsic module ISO_C_BINDING (18.2.3.3)`。
- **L3166 EN**: Continues the surrounding expression or declaration: `std::optional<SpecificCall>`.
  **L3166 CN**: 继续构造周围的表达式或声明：`std::optional<SpecificCall>`。
- **L3167 EN**: Continues logic associated with callable symbol `HandleC_F_Pointer`.
  **L3167 CN**: 继续与可调用符号 `HandleC_F_Pointer` 相关的逻辑。
- **L3168 EN**: Continues the surrounding expression or declaration: `ActualArguments &arguments, FoldingContext &context) const {`.
  **L3168 CN**: 继续构造周围的表达式或声明：`ActualArguments &arguments, FoldingContext &context) const {`。

### Lines 3169-3192

````cpp
  characteristics::Procedure::Attrs attrs;
  attrs.set(characteristics::Procedure::Attr::Subroutine);
  static const char *const keywords[]{
      "cptr", "fptr", "shape", "lower", nullptr};
  characteristics::DummyArguments dummies;
  if (CheckAndRearrangeArguments(arguments, context.messages(), keywords, 2)) {
    CHECK(arguments.size() == 4);
    if (const auto *expr{arguments[0].value().UnwrapExpr()}) {
      // General semantic checks will catch an actual argument that's not
      // scalar.
      if (auto type{expr->GetType()}) {
        if (type->category() != TypeCategory::Derived ||
            type->IsPolymorphic() ||
            (type->GetDerivedTypeSpec().typeSymbol().name() !=
                    "__builtin_c_ptr" &&
                type->GetDerivedTypeSpec().typeSymbol().name() !=
                    "__builtin_c_devptr")) {
          context.messages().Say(arguments[0]->sourceLocation(),
              "CPTR= argument to C_F_POINTER() must be a C_PTR"_err_en_US);
        }
        characteristics::DummyDataObject cptr{
            characteristics::TypeAndShape{*type}};
        cptr.intent = common::Intent::In;
        dummies.emplace_back("cptr"s, std::move(cptr));
````
- **L3169 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attrs attrs;`.
  **L3169 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attrs attrs;`。
- **L3170 EN**: Executes a call or declaration centered on `attrs.set`.
  **L3170 CN**: 执行以 `attrs.set` 为核心的调用或声明。
- **L3171 EN**: Continues the surrounding expression or declaration: `static const char *const keywords[]{`.
  **L3171 CN**: 继续构造周围的表达式或声明：`static const char *const keywords[]{`。
- **L3172 EN**: Executes a standalone statement or declaration: `"cptr", "fptr", "shape", "lower", nullptr};`.
  **L3172 CN**: 执行一条独立语句或声明：`"cptr", "fptr", "shape", "lower", nullptr};`。
- **L3173 EN**: Executes a standalone statement or declaration: `characteristics::DummyArguments dummies;`.
  **L3173 CN**: 执行一条独立语句或声明：`characteristics::DummyArguments dummies;`。
- **L3174 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3174 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3175 EN**: Executes a call or declaration centered on `CHECK`.
  **L3175 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L3176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3176 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3177 EN**: Comment explains nearby logic, intent, or metadata: `General semantic checks will catch an actual argument that's not`.
  **L3177 CN**: 注释说明附近代码的逻辑、意图或元数据：`General semantic checks will catch an actual argument that's not`。
- **L3178 EN**: Comment explains nearby logic, intent, or metadata: `scalar.`.
  **L3178 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalar.`。
- **L3179 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3179 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3180 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3180 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3181 EN**: Continues logic associated with callable symbol `IsPolymorphic`.
  **L3181 CN**: 继续与可调用符号 `IsPolymorphic` 相关的逻辑。
- **L3182 EN**: Continues logic associated with callable symbol `GetDerivedTypeSpec`.
  **L3182 CN**: 继续与可调用符号 `GetDerivedTypeSpec` 相关的逻辑。
- **L3183 EN**: Continues the surrounding expression or declaration: `"__builtin_c_ptr" &&`.
  **L3183 CN**: 继续构造周围的表达式或声明：`"__builtin_c_ptr" &&`。
- **L3184 EN**: Continues logic associated with callable symbol `GetDerivedTypeSpec`.
  **L3184 CN**: 继续与可调用符号 `GetDerivedTypeSpec` 相关的逻辑。
- **L3185 EN**: Continues the surrounding expression or declaration: `"__builtin_c_devptr")) {`.
  **L3185 CN**: 继续构造周围的表达式或声明：`"__builtin_c_devptr")) {`。
- **L3186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3186 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3187 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3187 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3188 EN**: Closes the current lexical scope or compound statement.
  **L3188 CN**: 结束当前词法作用域或复合语句块。
- **L3189 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject cptr{`.
  **L3189 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject cptr{`。
- **L3190 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape{*type}};`.
  **L3190 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape{*type}};`。
- **L3191 EN**: Executes a standalone statement or declaration: `cptr.intent = common::Intent::In;`.
  **L3191 CN**: 执行一条独立语句或声明：`cptr.intent = common::Intent::In;`。
- **L3192 EN**: Executes a call or declaration centered on `dummies.emplace_back`.
  **L3192 CN**: 执行以 `dummies.emplace_back` 为核心的调用或声明。

### Lines 3193-3216

````cpp
      }
    }
    if (const auto *expr{arguments[1].value().UnwrapExpr()}) {
      int fptrRank{expr->Rank()};
      auto at{arguments[1]->sourceLocation()};
      if (auto type{expr->GetType()}) {
        if (type->HasDeferredTypeParameter()) {
          context.messages().Say(at,
              "FPTR= argument to C_F_POINTER() may not have a deferred type parameter"_err_en_US);
        }
        if (ExtractCoarrayRef(*expr)) {
          context.messages().Say(at,
              "FPTR= argument to C_F_POINTER() may not be a coindexed object"_err_en_US);
        }
        characteristics::DummyDataObject fptr{
            characteristics::TypeAndShape{*type, fptrRank}};
        fptr.intent = common::Intent::Out;
        fptr.attrs.set(characteristics::DummyDataObject::Attr::Pointer);
        dummies.emplace_back("fptr"s, std::move(fptr));
      } else {
        context.messages().Say(
            at, "FPTR= argument to C_F_POINTER() must have a type"_err_en_US);
      }
      if (arguments[2] && fptrRank == 0) {
````
- **L3193 EN**: Closes the current lexical scope or compound statement.
  **L3193 CN**: 结束当前词法作用域或复合语句块。
- **L3194 EN**: Closes the current lexical scope or compound statement.
  **L3194 CN**: 结束当前词法作用域或复合语句块。
- **L3195 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3195 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3196 EN**: Executes a call or declaration centered on `fptrRank{expr->Rank`.
  **L3196 CN**: 执行以 `fptrRank{expr->Rank` 为核心的调用或声明。
- **L3197 EN**: Executes a call or declaration centered on `at{arguments[1]->sourceLocation`.
  **L3197 CN**: 执行以 `at{arguments[1]->sourceLocation` 为核心的调用或声明。
- **L3198 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3198 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3199 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3199 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3200 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3201 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3201 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3202 EN**: Closes the current lexical scope or compound statement.
  **L3202 CN**: 结束当前词法作用域或复合语句块。
- **L3203 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3203 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3204 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3205 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3205 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3206 EN**: Closes the current lexical scope or compound statement.
  **L3206 CN**: 结束当前词法作用域或复合语句块。
- **L3207 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject fptr{`.
  **L3207 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject fptr{`。
- **L3208 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape{*type, fptrRank}};`.
  **L3208 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape{*type, fptrRank}};`。
- **L3209 EN**: Executes a standalone statement or declaration: `fptr.intent = common::Intent::Out;`.
  **L3209 CN**: 执行一条独立语句或声明：`fptr.intent = common::Intent::Out;`。
- **L3210 EN**: Executes a call or declaration centered on `fptr.attrs.set`.
  **L3210 CN**: 执行以 `fptr.attrs.set` 为核心的调用或声明。
- **L3211 EN**: Executes a call or declaration centered on `dummies.emplace_back`.
  **L3211 CN**: 执行以 `dummies.emplace_back` 为核心的调用或声明。
- **L3212 EN**: Transitions from the previous branch into the alternative path.
  **L3212 CN**: 从前一个分支过渡到备选路径。
- **L3213 EN**: Continues logic associated with callable symbol `messages`.
  **L3213 CN**: 继续与可调用符号 `messages` 相关的逻辑。
- **L3214 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3214 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3215 EN**: Closes the current lexical scope or compound statement.
  **L3215 CN**: 结束当前词法作用域或复合语句块。
- **L3216 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3216 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3217-3240

````cpp
        context.messages().Say(arguments[2]->sourceLocation(),
            "SHAPE= argument to C_F_POINTER() may not appear when FPTR= is scalar"_err_en_US);
      } else if (!arguments[2] && fptrRank > 0) {
        context.messages().Say(
            "SHAPE= argument to C_F_POINTER() must appear when FPTR= is an array"_err_en_US);
      } else if (arguments[2]) {
        if (const auto *argExpr{arguments[2].value().UnwrapExpr()}) {
          if (argExpr->Rank() > 1) {
            context.messages().Say(arguments[2]->sourceLocation(),
                "SHAPE= argument to C_F_POINTER() must be a rank-one array."_err_en_US);
          } else if (argExpr->Rank() == 1) {
            if (auto constShape{GetConstantShape(context, *argExpr)}) {
              if (constShape->At(ConstantSubscripts{1}).ToInt64() != fptrRank) {
                context.messages().Say(arguments[2]->sourceLocation(),
                    "SHAPE= argument to C_F_POINTER() must have size equal to the rank of FPTR="_err_en_US);
              }
            }
          }
        }
      }
      if (arguments[3] && fptrRank == 0) {
        context.messages().Say(arguments[3]->sourceLocation(),
            "LOWER= argument to C_F_POINTER() may not appear when FPTR= is scalar"_err_en_US);
      } else if (arguments[3]) {
````
- **L3217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[2]->sourceLocation(),`.
  **L3217 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[2]->sourceLocation(),`。
- **L3218 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3218 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3219 EN**: Transitions from the previous branch into an `else if` condition.
  **L3219 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3220 EN**: Continues logic associated with callable symbol `messages`.
  **L3220 CN**: 继续与可调用符号 `messages` 相关的逻辑。
- **L3221 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3221 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3222 EN**: Transitions from the previous branch into an `else if` condition.
  **L3222 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3223 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3223 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3224 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3224 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[2]->sourceLocation(),`.
  **L3225 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[2]->sourceLocation(),`。
- **L3226 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3226 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3227 EN**: Transitions from the previous branch into an `else if` condition.
  **L3227 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3228 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3228 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3229 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3229 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[2]->sourceLocation(),`.
  **L3230 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[2]->sourceLocation(),`。
- **L3231 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3231 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3232 EN**: Closes the current lexical scope or compound statement.
  **L3232 CN**: 结束当前词法作用域或复合语句块。
- **L3233 EN**: Closes the current lexical scope or compound statement.
  **L3233 CN**: 结束当前词法作用域或复合语句块。
- **L3234 EN**: Closes the current lexical scope or compound statement.
  **L3234 CN**: 结束当前词法作用域或复合语句块。
- **L3235 EN**: Closes the current lexical scope or compound statement.
  **L3235 CN**: 结束当前词法作用域或复合语句块。
- **L3236 EN**: Closes the current lexical scope or compound statement.
  **L3236 CN**: 结束当前词法作用域或复合语句块。
- **L3237 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3237 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[3]->sourceLocation(),`.
  **L3238 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[3]->sourceLocation(),`。
- **L3239 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3239 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3240 EN**: Transitions from the previous branch into an `else if` condition.
  **L3240 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 3241-3264

````cpp
        if (const auto *argExpr{arguments[3].value().UnwrapExpr()}) {
          if (argExpr->Rank() > 1) {
            context.messages().Say(arguments[3]->sourceLocation(),
                "LOWER= argument to C_F_POINTER() must be a rank-one array."_err_en_US);
          } else if (argExpr->Rank() == 1) {
            if (auto constShape{GetConstantShape(context, *argExpr)}) {
              if (constShape->At(ConstantSubscripts{1}).ToInt64() != fptrRank) {
                context.messages().Say(arguments[3]->sourceLocation(),
                    "LOWER= argument to C_F_POINTER() must have size equal to the rank of FPTR="_err_en_US);
              }
            }
          }
        }
      }
    }
  }
  if (dummies.size() == 2) {
    // Handle SHAPE
    DynamicType shapeType{TypeCategory::Integer, defaults_.sizeIntegerKind()};
    if (arguments.size() >= 3 && arguments[2]) {
      if (auto type{arguments[2]->GetType()}) {
        if (type->category() == TypeCategory::Integer) {
          shapeType = *type;
        }
````
- **L3241 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3241 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3242 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3242 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[3]->sourceLocation(),`.
  **L3243 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[3]->sourceLocation(),`。
- **L3244 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3244 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3245 EN**: Transitions from the previous branch into an `else if` condition.
  **L3245 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3246 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3246 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3247 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3247 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[3]->sourceLocation(),`.
  **L3248 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[3]->sourceLocation(),`。
- **L3249 EN**: Executes a call or declaration centered on `C_F_POINTER`.
  **L3249 CN**: 执行以 `C_F_POINTER` 为核心的调用或声明。
- **L3250 EN**: Closes the current lexical scope or compound statement.
  **L3250 CN**: 结束当前词法作用域或复合语句块。
- **L3251 EN**: Closes the current lexical scope or compound statement.
  **L3251 CN**: 结束当前词法作用域或复合语句块。
- **L3252 EN**: Closes the current lexical scope or compound statement.
  **L3252 CN**: 结束当前词法作用域或复合语句块。
- **L3253 EN**: Closes the current lexical scope or compound statement.
  **L3253 CN**: 结束当前词法作用域或复合语句块。
- **L3254 EN**: Closes the current lexical scope or compound statement.
  **L3254 CN**: 结束当前词法作用域或复合语句块。
- **L3255 EN**: Closes the current lexical scope or compound statement.
  **L3255 CN**: 结束当前词法作用域或复合语句块。
- **L3256 EN**: Closes the current lexical scope or compound statement.
  **L3256 CN**: 结束当前词法作用域或复合语句块。
- **L3257 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3257 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3258 EN**: Comment explains nearby logic, intent, or metadata: `Handle SHAPE`.
  **L3258 CN**: 注释说明附近代码的逻辑、意图或元数据：`Handle SHAPE`。
- **L3259 EN**: Executes a call or declaration centered on `defaults_.sizeIntegerKind`.
  **L3259 CN**: 执行以 `defaults_.sizeIntegerKind` 为核心的调用或声明。
- **L3260 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3260 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3261 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3261 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3262 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3262 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3263 EN**: Executes a standalone statement or declaration: `shapeType = *type;`.
  **L3263 CN**: 执行一条独立语句或声明：`shapeType = *type;`。
- **L3264 EN**: Closes the current lexical scope or compound statement.
  **L3264 CN**: 结束当前词法作用域或复合语句块。

### Lines 3265-3288

````cpp
      }
    }
    characteristics::DummyDataObject shape{
        characteristics::TypeAndShape{shapeType, 1}};
    shape.intent = common::Intent::In;
    shape.attrs.set(characteristics::DummyDataObject::Attr::Optional);
    dummies.emplace_back("shape"s, std::move(shape));

    // Handle LOWER
    DynamicType lowerType{TypeCategory::Integer, defaults_.sizeIntegerKind()};
    if (arguments.size() >= 4 && arguments[3]) {
      if (auto type{arguments[3]->GetType()}) {
        if (type->category() == TypeCategory::Integer) {
          lowerType = *type;
        }
      }
    }
    characteristics::DummyDataObject lower{
        characteristics::TypeAndShape{lowerType, 1}};
    lower.intent = common::Intent::In;
    lower.attrs.set(characteristics::DummyDataObject::Attr::Optional);
    dummies.emplace_back("lower"s, std::move(lower));

    return SpecificCall{
````
- **L3265 EN**: Closes the current lexical scope or compound statement.
  **L3265 CN**: 结束当前词法作用域或复合语句块。
- **L3266 EN**: Closes the current lexical scope or compound statement.
  **L3266 CN**: 结束当前词法作用域或复合语句块。
- **L3267 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject shape{`.
  **L3267 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject shape{`。
- **L3268 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape{shapeType, 1}};`.
  **L3268 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape{shapeType, 1}};`。
- **L3269 EN**: Executes a standalone statement or declaration: `shape.intent = common::Intent::In;`.
  **L3269 CN**: 执行一条独立语句或声明：`shape.intent = common::Intent::In;`。
- **L3270 EN**: Executes a call or declaration centered on `shape.attrs.set`.
  **L3270 CN**: 执行以 `shape.attrs.set` 为核心的调用或声明。
- **L3271 EN**: Executes a call or declaration centered on `dummies.emplace_back`.
  **L3271 CN**: 执行以 `dummies.emplace_back` 为核心的调用或声明。
- **L3272 EN**: Blank line separating nearby declarations or logic blocks.
  **L3272 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3273 EN**: Comment explains nearby logic, intent, or metadata: `Handle LOWER`.
  **L3273 CN**: 注释说明附近代码的逻辑、意图或元数据：`Handle LOWER`。
- **L3274 EN**: Executes a call or declaration centered on `defaults_.sizeIntegerKind`.
  **L3274 CN**: 执行以 `defaults_.sizeIntegerKind` 为核心的调用或声明。
- **L3275 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3275 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3276 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3276 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3277 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3277 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3278 EN**: Executes a standalone statement or declaration: `lowerType = *type;`.
  **L3278 CN**: 执行一条独立语句或声明：`lowerType = *type;`。
- **L3279 EN**: Closes the current lexical scope or compound statement.
  **L3279 CN**: 结束当前词法作用域或复合语句块。
- **L3280 EN**: Closes the current lexical scope or compound statement.
  **L3280 CN**: 结束当前词法作用域或复合语句块。
- **L3281 EN**: Closes the current lexical scope or compound statement.
  **L3281 CN**: 结束当前词法作用域或复合语句块。
- **L3282 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject lower{`.
  **L3282 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject lower{`。
- **L3283 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape{lowerType, 1}};`.
  **L3283 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape{lowerType, 1}};`。
- **L3284 EN**: Executes a standalone statement or declaration: `lower.intent = common::Intent::In;`.
  **L3284 CN**: 执行一条独立语句或声明：`lower.intent = common::Intent::In;`。
- **L3285 EN**: Executes a call or declaration centered on `lower.attrs.set`.
  **L3285 CN**: 执行以 `lower.attrs.set` 为核心的调用或声明。
- **L3286 EN**: Executes a call or declaration centered on `dummies.emplace_back`.
  **L3286 CN**: 执行以 `dummies.emplace_back` 为核心的调用或声明。
- **L3287 EN**: Blank line separating nearby declarations or logic blocks.
  **L3287 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3288 EN**: Returns from the current function with `SpecificCall{`.
  **L3288 CN**: 以 `SpecificCall{` 从当前函数返回。

### Lines 3289-3312

````cpp
        SpecificIntrinsic{"__builtin_c_f_pointer"s,
            characteristics::Procedure{std::move(dummies), attrs}},
        std::move(arguments)};
  } else {
    return std::nullopt;
  }
}

// Subroutine C_F_STRPOINTER from intrinsic module ISO_C_BINDING (18.2.3.5)
// C_F_STRPOINTER(CSTRARRAY, FSTRPTR [,NCHARS]) or
// C_F_STRPOINTER(CSTRPTR, FSTRPTR, NCHARS)
std::optional<SpecificCall>
IntrinsicProcTable::Implementation::HandleC_F_Strpointer(
    ActualArguments &arguments, FoldingContext &context) const {
  characteristics::Procedure::Attrs attrs;
  attrs.set(characteristics::Procedure::Attr::Subroutine);

  // The first argument can be either CSTRARRAY or CSTRPTR (overloaded).
  // Assign common internal keyword "cstr" for CheckAndRearrangeArguments.
  std::optional<std::string> firstArgKeyword;
  for (auto &arg : arguments) {
    if (arg && arg->keyword()) {
      auto kw{arg->keyword()->ToString()};
      if (kw == "cstrarray" || kw == "cstrptr") {
````
- **L3289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SpecificIntrinsic{"__builtin_c_f_pointer"s,`.
  **L3289 CN**: 继续一个多行参数列表、初始化器或聚合项：`SpecificIntrinsic{"__builtin_c_f_pointer"s,`。
- **L3290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::Procedure{std::move(dummies), attrs}},`.
  **L3290 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::Procedure{std::move(dummies), attrs}},`。
- **L3291 EN**: Executes a call or declaration centered on `std::move`.
  **L3291 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3292 EN**: Transitions from the previous branch into the alternative path.
  **L3292 CN**: 从前一个分支过渡到备选路径。
- **L3293 EN**: Returns from the current function with `std::nullopt`.
  **L3293 CN**: 以 `std::nullopt` 从当前函数返回。
- **L3294 EN**: Closes the current lexical scope or compound statement.
  **L3294 CN**: 结束当前词法作用域或复合语句块。
- **L3295 EN**: Closes the current lexical scope or compound statement.
  **L3295 CN**: 结束当前词法作用域或复合语句块。
- **L3296 EN**: Blank line separating nearby declarations or logic blocks.
  **L3296 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3297 EN**: Comment explains nearby logic, intent, or metadata: `Subroutine C_F_STRPOINTER from intrinsic module ISO_C_BINDING (18.2.3.5)`.
  **L3297 CN**: 注释说明附近代码的逻辑、意图或元数据：`Subroutine C_F_STRPOINTER from intrinsic module ISO_C_BINDING (18.2.3.5)`。
- **L3298 EN**: Comment explains nearby logic, intent, or metadata: `C_F_STRPOINTER(CSTRARRAY, FSTRPTR [,NCHARS]) or`.
  **L3298 CN**: 注释说明附近代码的逻辑、意图或元数据：`C_F_STRPOINTER(CSTRARRAY, FSTRPTR [,NCHARS]) or`。
- **L3299 EN**: Comment explains nearby logic, intent, or metadata: `C_F_STRPOINTER(CSTRPTR, FSTRPTR, NCHARS)`.
  **L3299 CN**: 注释说明附近代码的逻辑、意图或元数据：`C_F_STRPOINTER(CSTRPTR, FSTRPTR, NCHARS)`。
- **L3300 EN**: Continues the surrounding expression or declaration: `std::optional<SpecificCall>`.
  **L3300 CN**: 继续构造周围的表达式或声明：`std::optional<SpecificCall>`。
- **L3301 EN**: Continues logic associated with callable symbol `HandleC_F_Strpointer`.
  **L3301 CN**: 继续与可调用符号 `HandleC_F_Strpointer` 相关的逻辑。
- **L3302 EN**: Continues the surrounding expression or declaration: `ActualArguments &arguments, FoldingContext &context) const {`.
  **L3302 CN**: 继续构造周围的表达式或声明：`ActualArguments &arguments, FoldingContext &context) const {`。
- **L3303 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attrs attrs;`.
  **L3303 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attrs attrs;`。
- **L3304 EN**: Executes a call or declaration centered on `attrs.set`.
  **L3304 CN**: 执行以 `attrs.set` 为核心的调用或声明。
- **L3305 EN**: Blank line separating nearby declarations or logic blocks.
  **L3305 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3306 EN**: Comment explains nearby logic, intent, or metadata: `The first argument can be either CSTRARRAY or CSTRPTR (overloaded).`.
  **L3306 CN**: 注释说明附近代码的逻辑、意图或元数据：`The first argument can be either CSTRARRAY or CSTRPTR (overloaded).`。
- **L3307 EN**: Comment explains nearby logic, intent, or metadata: `Assign common internal keyword "cstr" for CheckAndRearrangeArguments.`.
  **L3307 CN**: 注释说明附近代码的逻辑、意图或元数据：`Assign common internal keyword "cstr" for CheckAndRearrangeArguments.`。
- **L3308 EN**: Executes a standalone statement or declaration: `std::optional<std::string> firstArgKeyword;`.
  **L3308 CN**: 执行一条独立语句或声明：`std::optional<std::string> firstArgKeyword;`。
- **L3309 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3309 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3310 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3310 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3311 EN**: Executes a call or declaration centered on `kw{arg->keyword`.
  **L3311 CN**: 执行以 `kw{arg->keyword` 为核心的调用或声明。
- **L3312 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3312 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3313-3336

````cpp
        if (!firstArgKeyword) {
          firstArgKeyword = kw;
        }
        static const char cstrKeyword[] = "cstr";
        arg->set_keyword(
            parser::CharBlock{cstrKeyword, sizeof(cstrKeyword) - 1});
      }
    }
  }

  static const char *const keywords[]{"cstr", "fstrptr", "nchars", nullptr};
  characteristics::DummyArguments dummies;
  if (CheckAndRearrangeArguments(arguments, context.messages(), keywords, 1)) {
    CHECK(arguments.size() == 3);
    const bool hasNchars{arguments[2].has_value()};
    const int cCharKind = defaults_.GetDefaultKind(TypeCategory::Character);

    // Check first argument (CSTRARRAY or CSTRPTR) and optional third argument
    // (NCHARS)
    if (const auto *expr{arguments[0].value().UnwrapExpr()}) {
      const auto at{arguments[0]->sourceLocation()};
      if (const auto type{expr->GetType()}) {
        if (type->category() == TypeCategory::Derived &&
            !type->IsPolymorphic() &&
````
- **L3313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3314 EN**: Executes a standalone statement or declaration: `firstArgKeyword = kw;`.
  **L3314 CN**: 执行一条独立语句或声明：`firstArgKeyword = kw;`。
- **L3315 EN**: Closes the current lexical scope or compound statement.
  **L3315 CN**: 结束当前词法作用域或复合语句块。
- **L3316 EN**: Executes a standalone statement or declaration: `static const char cstrKeyword[] = "cstr";`.
  **L3316 CN**: 执行一条独立语句或声明：`static const char cstrKeyword[] = "cstr";`。
- **L3317 EN**: Continues logic associated with callable symbol `set_keyword`.
  **L3317 CN**: 继续与可调用符号 `set_keyword` 相关的逻辑。
- **L3318 EN**: Executes a call or declaration centered on `sizeof`.
  **L3318 CN**: 执行以 `sizeof` 为核心的调用或声明。
- **L3319 EN**: Closes the current lexical scope or compound statement.
  **L3319 CN**: 结束当前词法作用域或复合语句块。
- **L3320 EN**: Closes the current lexical scope or compound statement.
  **L3320 CN**: 结束当前词法作用域或复合语句块。
- **L3321 EN**: Closes the current lexical scope or compound statement.
  **L3321 CN**: 结束当前词法作用域或复合语句块。
- **L3322 EN**: Blank line separating nearby declarations or logic blocks.
  **L3322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3323 EN**: Executes a standalone statement or declaration: `static const char *const keywords[]{"cstr", "fstrptr", "nchars", nullptr};`.
  **L3323 CN**: 执行一条独立语句或声明：`static const char *const keywords[]{"cstr", "fstrptr", "nchars", nullptr};`。
- **L3324 EN**: Executes a standalone statement or declaration: `characteristics::DummyArguments dummies;`.
  **L3324 CN**: 执行一条独立语句或声明：`characteristics::DummyArguments dummies;`。
- **L3325 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3325 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3326 EN**: Executes a call or declaration centered on `CHECK`.
  **L3326 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L3327 EN**: Executes a call or declaration centered on `hasNchars{arguments[2].has_value`.
  **L3327 CN**: 执行以 `hasNchars{arguments[2].has_value` 为核心的调用或声明。
- **L3328 EN**: Initializes variable `cCharKind` from the right-hand expression.
  **L3328 CN**: 使用右侧表达式初始化变量 `cCharKind`。
- **L3329 EN**: Blank line separating nearby declarations or logic blocks.
  **L3329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3330 EN**: Comment explains nearby logic, intent, or metadata: `Check first argument (CSTRARRAY or CSTRPTR) and optional third argument`.
  **L3330 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check first argument (CSTRARRAY or CSTRPTR) and optional third argument`。
- **L3331 EN**: Comment explains nearby logic, intent, or metadata: `(NCHARS)`.
  **L3331 CN**: 注释说明附近代码的逻辑、意图或元数据：`(NCHARS)`。
- **L3332 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3332 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3333 EN**: Executes a call or declaration centered on `at{arguments[0]->sourceLocation`.
  **L3333 CN**: 执行以 `at{arguments[0]->sourceLocation` 为核心的调用或声明。
- **L3334 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3334 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3335 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3335 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3336 EN**: Continues logic associated with callable symbol `IsPolymorphic`.
  **L3336 CN**: 继续与可调用符号 `IsPolymorphic` 相关的逻辑。

### Lines 3337-3360

````cpp
            (type->GetDerivedTypeSpec().typeSymbol().name() ==
                    "__builtin_c_ptr" ||
                type->GetDerivedTypeSpec().typeSymbol().name() ==
                    "__builtin_c_devptr")) {
          // First argument is C_PTR (CSTRPTR form)
          if (firstArgKeyword && *firstArgKeyword != "cstrptr") {
            context.messages().Say(at,
                "Keyword CSTRARRAY= cannot be used with a C_PTR argument; use CSTRPTR= instead"_err_en_US);
          }
          if (!hasNchars) {
            context.messages().Say(at,
                "NCHARS= argument is required when CSTRPTR= appears in C_F_STRPOINTER()"_err_en_US);
          }
          characteristics::DummyDataObject cstrptr{
              characteristics::TypeAndShape{*type}};
          cstrptr.intent = common::Intent::In;
          dummies.emplace_back("cstrptr"s, std::move(cstrptr));
        } else if (type->category() == TypeCategory::Character) {
          // First argument should be CSTRARRAY - rank-1 character array
          if (firstArgKeyword && *firstArgKeyword != "cstrarray") {
            context.messages().Say(at,
                "Keyword CSTRPTR= cannot be used with a character array argument; use CSTRARRAY= instead"_err_en_US);
          }
          if (type->kind() != cCharKind) {
````
- **L3337 EN**: Continues logic associated with callable symbol `GetDerivedTypeSpec`.
  **L3337 CN**: 继续与可调用符号 `GetDerivedTypeSpec` 相关的逻辑。
- **L3338 EN**: Continues the surrounding expression or declaration: `"__builtin_c_ptr" ||`.
  **L3338 CN**: 继续构造周围的表达式或声明：`"__builtin_c_ptr" ||`。
- **L3339 EN**: Continues logic associated with callable symbol `GetDerivedTypeSpec`.
  **L3339 CN**: 继续与可调用符号 `GetDerivedTypeSpec` 相关的逻辑。
- **L3340 EN**: Continues the surrounding expression or declaration: `"__builtin_c_devptr")) {`.
  **L3340 CN**: 继续构造周围的表达式或声明：`"__builtin_c_devptr")) {`。
- **L3341 EN**: Comment explains nearby logic, intent, or metadata: `First argument is C_PTR (CSTRPTR form)`.
  **L3341 CN**: 注释说明附近代码的逻辑、意图或元数据：`First argument is C_PTR (CSTRPTR form)`。
- **L3342 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3342 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3343 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3344 EN**: Executes a standalone statement or declaration: `"Keyword CSTRARRAY= cannot be used with a C_PTR argument; use CSTRPTR= instead"_err_en_US);`.
  **L3344 CN**: 执行一条独立语句或声明：`"Keyword CSTRARRAY= cannot be used with a C_PTR argument; use CSTRPTR= instead"_err_en_US);`。
- **L3345 EN**: Closes the current lexical scope or compound statement.
  **L3345 CN**: 结束当前词法作用域或复合语句块。
- **L3346 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3346 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3347 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3348 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3348 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3349 EN**: Closes the current lexical scope or compound statement.
  **L3349 CN**: 结束当前词法作用域或复合语句块。
- **L3350 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject cstrptr{`.
  **L3350 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject cstrptr{`。
- **L3351 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape{*type}};`.
  **L3351 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape{*type}};`。
- **L3352 EN**: Executes a standalone statement or declaration: `cstrptr.intent = common::Intent::In;`.
  **L3352 CN**: 执行一条独立语句或声明：`cstrptr.intent = common::Intent::In;`。
- **L3353 EN**: Executes a call or declaration centered on `dummies.emplace_back`.
  **L3353 CN**: 执行以 `dummies.emplace_back` 为核心的调用或声明。
- **L3354 EN**: Transitions from the previous branch into an `else if` condition.
  **L3354 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3355 EN**: Comment explains nearby logic, intent, or metadata: `First argument should be CSTRARRAY - rank-1 character array`.
  **L3355 CN**: 注释说明附近代码的逻辑、意图或元数据：`First argument should be CSTRARRAY - rank-1 character array`。
- **L3356 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3356 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3357 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3358 EN**: Executes a standalone statement or declaration: `"Keyword CSTRPTR= cannot be used with a character array argument; use CSTRARRAY= instead"_err_en_US);`.
  **L3358 CN**: 执行一条独立语句或声明：`"Keyword CSTRPTR= cannot be used with a character array argument; use CSTRARRAY= instead"_err_en_US);`。
- **L3359 EN**: Closes the current lexical scope or compound statement.
  **L3359 CN**: 结束当前词法作用域或复合语句块。
- **L3360 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3360 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3361-3384

````cpp
            context.messages().Say(at,
                "CSTRARRAY= argument to C_F_STRPOINTER() must be of kind C_CHAR"_err_en_US);
          }
          if (expr->Rank() != 1) {
            context.messages().Say(at,
                "CSTRARRAY= argument to C_F_STRPOINTER() must be a rank-one array"_err_en_US);
          }
          if (const auto len{type->GetCharLength()}) {
            if (const auto constLen{ToInt64(*len)}) {
              if (*constLen != 1) {
                context.messages().Say(at,
                    "CSTRARRAY= argument to C_F_STRPOINTER() must have length type parameter equal to one"_err_en_US);
              }
            }
          }
          // Check if CSTRARRAY is assumed-size and NCHARS is absent
          if (auto shape{GetShape(context, *expr)}) {
            if (shape->size() == 1) {
              const auto &extentExpr{(*shape)[0]};
              const auto extentInt{ToInt64(extentExpr)};
              if ((!extentInt || *extentInt < 0) && !hasNchars) {
                context.messages().Say(at,
                    "NCHARS= argument is required when CSTRARRAY= is assumed-size in C_F_STRPOINTER()"_err_en_US);
              }
````
- **L3361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3361 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3362 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3362 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3363 EN**: Closes the current lexical scope or compound statement.
  **L3363 CN**: 结束当前词法作用域或复合语句块。
- **L3364 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3364 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3365 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3366 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3366 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3367 EN**: Closes the current lexical scope or compound statement.
  **L3367 CN**: 结束当前词法作用域或复合语句块。
- **L3368 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3368 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3369 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3369 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3370 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3370 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3371 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3372 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3372 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3373 EN**: Closes the current lexical scope or compound statement.
  **L3373 CN**: 结束当前词法作用域或复合语句块。
- **L3374 EN**: Closes the current lexical scope or compound statement.
  **L3374 CN**: 结束当前词法作用域或复合语句块。
- **L3375 EN**: Closes the current lexical scope or compound statement.
  **L3375 CN**: 结束当前词法作用域或复合语句块。
- **L3376 EN**: Comment explains nearby logic, intent, or metadata: `Check if CSTRARRAY is assumed-size and NCHARS is absent`.
  **L3376 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if CSTRARRAY is assumed-size and NCHARS is absent`。
- **L3377 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3377 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3378 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3378 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3379 EN**: Executes a call or declaration centered on `&extentExpr{`.
  **L3379 CN**: 执行以 `&extentExpr{` 为核心的调用或声明。
- **L3380 EN**: Executes a call or declaration centered on `extentInt{ToInt64`.
  **L3380 CN**: 执行以 `extentInt{ToInt64` 为核心的调用或声明。
- **L3381 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3381 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3382 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3383 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3383 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3384 EN**: Closes the current lexical scope or compound statement.
  **L3384 CN**: 结束当前词法作用域或复合语句块。

### Lines 3385-3408

````cpp
            }
          }
          // Check if NCHARS > size(CSTRARRAY) at compile time
          if (hasNchars) {
            if (const auto *ncharsExpr{arguments[2]->UnwrapExpr()}) {
              if (const auto ncharsVal{ToInt64(*ncharsExpr)}) {
                if (const auto shape{GetShape(context, *expr)};
                    shape && shape->size() == 1) {
                  if (const auto arraySize{ToInt64((*shape)[0])};
                      arraySize && *arraySize > 0 && *ncharsVal > *arraySize) {
                    context.messages().Say(arguments[2]->sourceLocation(),
                        "NCHARS=%jd is greater than the size of CSTRARRAY=%jd in C_F_STRPOINTER()"_err_en_US,
                        static_cast<std::intmax_t>(*ncharsVal),
                        static_cast<std::intmax_t>(*arraySize));
                  }
                }
              }
            }
          }
          characteristics::DummyDataObject cstrarray{
              characteristics::TypeAndShape{*type, 1}};
          cstrarray.intent = common::Intent::In;
          cstrarray.attrs.set(characteristics::DummyDataObject::Attr::Target);
          dummies.emplace_back("cstrarray"s, std::move(cstrarray));
````
- **L3385 EN**: Closes the current lexical scope or compound statement.
  **L3385 CN**: 结束当前词法作用域或复合语句块。
- **L3386 EN**: Closes the current lexical scope or compound statement.
  **L3386 CN**: 结束当前词法作用域或复合语句块。
- **L3387 EN**: Comment explains nearby logic, intent, or metadata: `Check if NCHARS > size(CSTRARRAY) at compile time`.
  **L3387 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if NCHARS > size(CSTRARRAY) at compile time`。
- **L3388 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3388 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3389 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3389 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3390 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3390 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3391 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3391 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3392 EN**: Starts a function, method, lambda, or structured scope: `shape && shape->size() == 1) {`.
  **L3392 CN**: 开始一个函数、方法、lambda 或结构化作用域：`shape && shape->size() == 1) {`。
- **L3393 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3393 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3394 EN**: Continues the surrounding expression or declaration: `arraySize && *arraySize > 0 && *ncharsVal > *arraySize) {`.
  **L3394 CN**: 继续构造周围的表达式或声明：`arraySize && *arraySize > 0 && *ncharsVal > *arraySize) {`。
- **L3395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[2]->sourceLocation(),`.
  **L3395 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[2]->sourceLocation(),`。
- **L3396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"NCHARS=%jd is greater than the size of CSTRARRAY=%jd in C_F_STRPOINTER()"_err_en_US,`.
  **L3396 CN**: 继续一个多行参数列表、初始化器或聚合项：`"NCHARS=%jd is greater than the size of CSTRARRAY=%jd in C_F_STRPOINTER()"_err_en_US,`。
- **L3397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(*ncharsVal),`.
  **L3397 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(*ncharsVal),`。
- **L3398 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L3398 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L3399 EN**: Closes the current lexical scope or compound statement.
  **L3399 CN**: 结束当前词法作用域或复合语句块。
- **L3400 EN**: Closes the current lexical scope or compound statement.
  **L3400 CN**: 结束当前词法作用域或复合语句块。
- **L3401 EN**: Closes the current lexical scope or compound statement.
  **L3401 CN**: 结束当前词法作用域或复合语句块。
- **L3402 EN**: Closes the current lexical scope or compound statement.
  **L3402 CN**: 结束当前词法作用域或复合语句块。
- **L3403 EN**: Closes the current lexical scope or compound statement.
  **L3403 CN**: 结束当前词法作用域或复合语句块。
- **L3404 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject cstrarray{`.
  **L3404 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject cstrarray{`。
- **L3405 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape{*type, 1}};`.
  **L3405 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape{*type, 1}};`。
- **L3406 EN**: Executes a standalone statement or declaration: `cstrarray.intent = common::Intent::In;`.
  **L3406 CN**: 执行一条独立语句或声明：`cstrarray.intent = common::Intent::In;`。
- **L3407 EN**: Executes a call or declaration centered on `cstrarray.attrs.set`.
  **L3407 CN**: 执行以 `cstrarray.attrs.set` 为核心的调用或声明。
- **L3408 EN**: Executes a call or declaration centered on `dummies.emplace_back`.
  **L3408 CN**: 执行以 `dummies.emplace_back` 为核心的调用或声明。

### Lines 3409-3432

````cpp
        } else {
          context.messages().Say(at,
              "First argument to C_F_STRPOINTER() must be a C_PTR or a rank-one character array of kind C_CHAR"_err_en_US);
        }
      }
    }

    // Check FSTRPTR argument - must be scalar deferred-length character pointer
    if (const auto *expr{arguments[1].value().UnwrapExpr()}) {
      const auto at{arguments[1]->sourceLocation()};
      if (const auto type{expr->GetType()}) {
        if (type->category() != TypeCategory::Character) {
          context.messages().Say(at,
              "FSTRPTR= argument to C_F_STRPOINTER() must be a character pointer"_err_en_US);
        } else {
          if (type->kind() != cCharKind) {
            context.messages().Say(at,
                "FSTRPTR= argument to C_F_STRPOINTER() must be of kind C_CHAR"_err_en_US);
          }
          if (!type->HasDeferredTypeParameter()) {
            context.messages().Say(at,
                "FSTRPTR= argument to C_F_STRPOINTER() must have deferred length"_err_en_US);
          }
        }
````
- **L3409 EN**: Transitions from the previous branch into the alternative path.
  **L3409 CN**: 从前一个分支过渡到备选路径。
- **L3410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3410 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3411 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3411 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3412 EN**: Closes the current lexical scope or compound statement.
  **L3412 CN**: 结束当前词法作用域或复合语句块。
- **L3413 EN**: Closes the current lexical scope or compound statement.
  **L3413 CN**: 结束当前词法作用域或复合语句块。
- **L3414 EN**: Closes the current lexical scope or compound statement.
  **L3414 CN**: 结束当前词法作用域或复合语句块。
- **L3415 EN**: Blank line separating nearby declarations or logic blocks.
  **L3415 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3416 EN**: Comment explains nearby logic, intent, or metadata: `Check FSTRPTR argument - must be scalar deferred-length character pointer`.
  **L3416 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check FSTRPTR argument - must be scalar deferred-length character pointer`。
- **L3417 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3417 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3418 EN**: Executes a call or declaration centered on `at{arguments[1]->sourceLocation`.
  **L3418 CN**: 执行以 `at{arguments[1]->sourceLocation` 为核心的调用或声明。
- **L3419 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3419 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3420 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3420 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3421 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3422 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3422 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3423 EN**: Transitions from the previous branch into the alternative path.
  **L3423 CN**: 从前一个分支过渡到备选路径。
- **L3424 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3424 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3425 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3426 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3426 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3427 EN**: Closes the current lexical scope or compound statement.
  **L3427 CN**: 结束当前词法作用域或复合语句块。
- **L3428 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3428 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3429 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3430 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3430 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3431 EN**: Closes the current lexical scope or compound statement.
  **L3431 CN**: 结束当前词法作用域或复合语句块。
- **L3432 EN**: Closes the current lexical scope or compound statement.
  **L3432 CN**: 结束当前词法作用域或复合语句块。

### Lines 3433-3456

````cpp
        if (ExtractCoarrayRef(*expr)) {
          context.messages().Say(at,
              "FSTRPTR= argument to C_F_STRPOINTER() may not be a coindexed object"_err_en_US);
        }
        characteristics::DummyDataObject fstrptr{
            characteristics::TypeAndShape{*type, 0}};
        fstrptr.intent = common::Intent::Out;
        fstrptr.attrs.set(characteristics::DummyDataObject::Attr::Pointer);
        dummies.emplace_back("fstrptr"s, std::move(fstrptr));
      } else {
        context.messages().Say(at,
            "FSTRPTR= argument to C_F_STRPOINTER() must have a type"_err_en_US);
      }
    }

    // Check NCHARS argument if present
    if (hasNchars) {
      if (const auto *expr{arguments[2].value().UnwrapExpr()}) {
        const auto at{arguments[2]->sourceLocation()};
        if (const auto type{expr->GetType()}) {
          if (type->category() != TypeCategory::Integer) {
            context.messages().Say(at,
                "NCHARS= argument to C_F_STRPOINTER() must be an integer"_err_en_US);
          }
````
- **L3433 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3433 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3434 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3435 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3435 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3436 EN**: Closes the current lexical scope or compound statement.
  **L3436 CN**: 结束当前词法作用域或复合语句块。
- **L3437 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject fstrptr{`.
  **L3437 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject fstrptr{`。
- **L3438 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape{*type, 0}};`.
  **L3438 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape{*type, 0}};`。
- **L3439 EN**: Executes a standalone statement or declaration: `fstrptr.intent = common::Intent::Out;`.
  **L3439 CN**: 执行一条独立语句或声明：`fstrptr.intent = common::Intent::Out;`。
- **L3440 EN**: Executes a call or declaration centered on `fstrptr.attrs.set`.
  **L3440 CN**: 执行以 `fstrptr.attrs.set` 为核心的调用或声明。
- **L3441 EN**: Executes a call or declaration centered on `dummies.emplace_back`.
  **L3441 CN**: 执行以 `dummies.emplace_back` 为核心的调用或声明。
- **L3442 EN**: Transitions from the previous branch into the alternative path.
  **L3442 CN**: 从前一个分支过渡到备选路径。
- **L3443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3443 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3444 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3444 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3445 EN**: Closes the current lexical scope or compound statement.
  **L3445 CN**: 结束当前词法作用域或复合语句块。
- **L3446 EN**: Closes the current lexical scope or compound statement.
  **L3446 CN**: 结束当前词法作用域或复合语句块。
- **L3447 EN**: Blank line separating nearby declarations or logic blocks.
  **L3447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3448 EN**: Comment explains nearby logic, intent, or metadata: `Check NCHARS argument if present`.
  **L3448 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check NCHARS argument if present`。
- **L3449 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3449 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3450 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3450 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3451 EN**: Executes a call or declaration centered on `at{arguments[2]->sourceLocation`.
  **L3451 CN**: 执行以 `at{arguments[2]->sourceLocation` 为核心的调用或声明。
- **L3452 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3452 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3453 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3453 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3454 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3455 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3455 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3456 EN**: Closes the current lexical scope or compound statement.
  **L3456 CN**: 结束当前词法作用域或复合语句块。

### Lines 3457-3480

````cpp
        }
        if (expr->Rank() != 0) {
          context.messages().Say(at,
              "NCHARS= argument to C_F_STRPOINTER() must be a scalar"_err_en_US);
        }
        // Check for negative value if constant
        if (const auto ncharsVal{ToInt64(*expr)}) {
          if (*ncharsVal < 0) {
            context.messages().Say(at,
                "NCHARS= argument to C_F_STRPOINTER() must be non-negative"_err_en_US);
          }
        }
      }
    }
  }
  if (dummies.size() == 2) {
    // Add NCHARS dummy
    DynamicType ncharsType{TypeCategory::Integer, defaults_.sizeIntegerKind()};
    if (arguments.size() >= 3 && arguments[2]) {
      if (const auto type{arguments[2]->GetType()}) {
        if (type->category() == TypeCategory::Integer) {
          ncharsType = *type;
        }
      }
````
- **L3457 EN**: Closes the current lexical scope or compound statement.
  **L3457 CN**: 结束当前词法作用域或复合语句块。
- **L3458 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3458 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3459 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3460 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3460 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3461 EN**: Closes the current lexical scope or compound statement.
  **L3461 CN**: 结束当前词法作用域或复合语句块。
- **L3462 EN**: Comment explains nearby logic, intent, or metadata: `Check for negative value if constant`.
  **L3462 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check for negative value if constant`。
- **L3463 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3463 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3464 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3464 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(at,`.
  **L3465 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(at,`。
- **L3466 EN**: Executes a call or declaration centered on `C_F_STRPOINTER`.
  **L3466 CN**: 执行以 `C_F_STRPOINTER` 为核心的调用或声明。
- **L3467 EN**: Closes the current lexical scope or compound statement.
  **L3467 CN**: 结束当前词法作用域或复合语句块。
- **L3468 EN**: Closes the current lexical scope or compound statement.
  **L3468 CN**: 结束当前词法作用域或复合语句块。
- **L3469 EN**: Closes the current lexical scope or compound statement.
  **L3469 CN**: 结束当前词法作用域或复合语句块。
- **L3470 EN**: Closes the current lexical scope or compound statement.
  **L3470 CN**: 结束当前词法作用域或复合语句块。
- **L3471 EN**: Closes the current lexical scope or compound statement.
  **L3471 CN**: 结束当前词法作用域或复合语句块。
- **L3472 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3472 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3473 EN**: Comment explains nearby logic, intent, or metadata: `Add NCHARS dummy`.
  **L3473 CN**: 注释说明附近代码的逻辑、意图或元数据：`Add NCHARS dummy`。
- **L3474 EN**: Executes a call or declaration centered on `defaults_.sizeIntegerKind`.
  **L3474 CN**: 执行以 `defaults_.sizeIntegerKind` 为核心的调用或声明。
- **L3475 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3475 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3476 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3476 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3477 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3477 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3478 EN**: Executes a standalone statement or declaration: `ncharsType = *type;`.
  **L3478 CN**: 执行一条独立语句或声明：`ncharsType = *type;`。
- **L3479 EN**: Closes the current lexical scope or compound statement.
  **L3479 CN**: 结束当前词法作用域或复合语句块。
- **L3480 EN**: Closes the current lexical scope or compound statement.
  **L3480 CN**: 结束当前词法作用域或复合语句块。

### Lines 3481-3504

````cpp
    }
    characteristics::DummyDataObject nchars{
        characteristics::TypeAndShape{ncharsType}};
    nchars.intent = common::Intent::In;
    nchars.attrs.set(characteristics::DummyDataObject::Attr::Optional);
    dummies.emplace_back("nchars"s, std::move(nchars));

    return SpecificCall{
        SpecificIntrinsic{"__builtin_c_f_strpointer"s,
            characteristics::Procedure{std::move(dummies), attrs}},
        std::move(arguments)};
  } else {
    return std::nullopt;
  }
}

// Function C_LOC(X) from intrinsic module ISO_C_BINDING (18.2.3.6)
std::optional<SpecificCall> IntrinsicProcTable::Implementation::HandleC_Loc(
    ActualArguments &arguments, FoldingContext &context) const {
  static const char *const keywords[]{"x", nullptr};
  if (CheckAndRearrangeArguments(arguments, context.messages(), keywords)) {
    CHECK(arguments.size() == 1);
    CheckForCoindexedObject(context.messages(), arguments[0], "c_loc", "x");
    const auto *expr{arguments[0].value().UnwrapExpr()};
````
- **L3481 EN**: Closes the current lexical scope or compound statement.
  **L3481 CN**: 结束当前词法作用域或复合语句块。
- **L3482 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject nchars{`.
  **L3482 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject nchars{`。
- **L3483 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape{ncharsType}};`.
  **L3483 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape{ncharsType}};`。
- **L3484 EN**: Executes a standalone statement or declaration: `nchars.intent = common::Intent::In;`.
  **L3484 CN**: 执行一条独立语句或声明：`nchars.intent = common::Intent::In;`。
- **L3485 EN**: Executes a call or declaration centered on `nchars.attrs.set`.
  **L3485 CN**: 执行以 `nchars.attrs.set` 为核心的调用或声明。
- **L3486 EN**: Executes a call or declaration centered on `dummies.emplace_back`.
  **L3486 CN**: 执行以 `dummies.emplace_back` 为核心的调用或声明。
- **L3487 EN**: Blank line separating nearby declarations or logic blocks.
  **L3487 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3488 EN**: Returns from the current function with `SpecificCall{`.
  **L3488 CN**: 以 `SpecificCall{` 从当前函数返回。
- **L3489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SpecificIntrinsic{"__builtin_c_f_strpointer"s,`.
  **L3489 CN**: 继续一个多行参数列表、初始化器或聚合项：`SpecificIntrinsic{"__builtin_c_f_strpointer"s,`。
- **L3490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::Procedure{std::move(dummies), attrs}},`.
  **L3490 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::Procedure{std::move(dummies), attrs}},`。
- **L3491 EN**: Executes a call or declaration centered on `std::move`.
  **L3491 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3492 EN**: Transitions from the previous branch into the alternative path.
  **L3492 CN**: 从前一个分支过渡到备选路径。
- **L3493 EN**: Returns from the current function with `std::nullopt`.
  **L3493 CN**: 以 `std::nullopt` 从当前函数返回。
- **L3494 EN**: Closes the current lexical scope or compound statement.
  **L3494 CN**: 结束当前词法作用域或复合语句块。
- **L3495 EN**: Closes the current lexical scope or compound statement.
  **L3495 CN**: 结束当前词法作用域或复合语句块。
- **L3496 EN**: Blank line separating nearby declarations or logic blocks.
  **L3496 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3497 EN**: Comment explains nearby logic, intent, or metadata: `Function C_LOC(X) from intrinsic module ISO_C_BINDING (18.2.3.6)`.
  **L3497 CN**: 注释说明附近代码的逻辑、意图或元数据：`Function C_LOC(X) from intrinsic module ISO_C_BINDING (18.2.3.6)`。
- **L3498 EN**: Continues logic associated with callable symbol `HandleC_Loc`.
  **L3498 CN**: 继续与可调用符号 `HandleC_Loc` 相关的逻辑。
- **L3499 EN**: Continues the surrounding expression or declaration: `ActualArguments &arguments, FoldingContext &context) const {`.
  **L3499 CN**: 继续构造周围的表达式或声明：`ActualArguments &arguments, FoldingContext &context) const {`。
- **L3500 EN**: Executes a standalone statement or declaration: `static const char *const keywords[]{"x", nullptr};`.
  **L3500 CN**: 执行一条独立语句或声明：`static const char *const keywords[]{"x", nullptr};`。
- **L3501 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3501 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3502 EN**: Executes a call or declaration centered on `CHECK`.
  **L3502 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L3503 EN**: Executes a call or declaration centered on `CheckForCoindexedObject`.
  **L3503 CN**: 执行以 `CheckForCoindexedObject` 为核心的调用或声明。
- **L3504 EN**: Executes a call or declaration centered on `*expr{arguments[0].value`.
  **L3504 CN**: 执行以 `*expr{arguments[0].value` 为核心的调用或声明。

### Lines 3505-3528

````cpp
    SpecificCall specificCall{
        SpecificIntrinsic{"__builtin_c_loc"s,
            characteristics::Procedure{
                characteristics::FunctionResult{DynamicType{
                    GetBuiltinDerivedType(builtinsScope_, "__builtin_c_ptr")}},
                characteristics::DummyArguments{},
                characteristics::Procedure::Attrs{
                    characteristics::Procedure::Attr::Pure}}},
        {/*arguments*/}};
    if (expr &&
        !(IsObjectPointer(*expr) ||
            (IsVariable(*expr) && GetLastTarget(GetSymbolVector(*expr))))) {
      if (context.languageFeatures().IsEnabled(
              common::LanguageFeature::RelaxedCLoc)) {
        context.Warn(common::UsageWarning::CLoc, arguments[0]->sourceLocation(),
            "C_LOC() argument should be a data pointer or target"_warn_en_US);
      } else {
        context.messages().Say(arguments[0]->sourceLocation(),
            "C_LOC() argument must be a data pointer or target"_err_en_US);
      }
    }
    if (auto typeAndShape{characteristics::TypeAndShape::Characterize(
            arguments[0], context)}) {
      if (expr && !IsContiguous(*expr, context).value_or(true)) {
````
- **L3505 EN**: Continues the surrounding expression or declaration: `SpecificCall specificCall{`.
  **L3505 CN**: 继续构造周围的表达式或声明：`SpecificCall specificCall{`。
- **L3506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SpecificIntrinsic{"__builtin_c_loc"s,`.
  **L3506 CN**: 继续一个多行参数列表、初始化器或聚合项：`SpecificIntrinsic{"__builtin_c_loc"s,`。
- **L3507 EN**: Continues the surrounding expression or declaration: `characteristics::Procedure{`.
  **L3507 CN**: 继续构造周围的表达式或声明：`characteristics::Procedure{`。
- **L3508 EN**: Continues the surrounding expression or declaration: `characteristics::FunctionResult{DynamicType{`.
  **L3508 CN**: 继续构造周围的表达式或声明：`characteristics::FunctionResult{DynamicType{`。
- **L3509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `GetBuiltinDerivedType(builtinsScope_, "__builtin_c_ptr")}},`.
  **L3509 CN**: 继续一个多行参数列表、初始化器或聚合项：`GetBuiltinDerivedType(builtinsScope_, "__builtin_c_ptr")}},`。
- **L3510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyArguments{},`.
  **L3510 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyArguments{},`。
- **L3511 EN**: Continues the surrounding expression or declaration: `characteristics::Procedure::Attrs{`.
  **L3511 CN**: 继续构造周围的表达式或声明：`characteristics::Procedure::Attrs{`。
- **L3512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::Procedure::Attr::Pure}}},`.
  **L3512 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::Procedure::Attr::Pure}}},`。
- **L3513 EN**: Executes a standalone statement or declaration: `{/*arguments*/}};`.
  **L3513 CN**: 执行一条独立语句或声明：`{/*arguments*/}};`。
- **L3514 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3514 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3515 EN**: Continues logic associated with callable symbol `IsObjectPointer`.
  **L3515 CN**: 继续与可调用符号 `IsObjectPointer` 相关的逻辑。
- **L3516 EN**: Starts a function, method, lambda, or structured scope: `(IsVariable(*expr) && GetLastTarget(GetSymbolVector(*expr))))) {`.
  **L3516 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(IsVariable(*expr) && GetLastTarget(GetSymbolVector(*expr))))) {`。
- **L3517 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3517 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3518 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::RelaxedCLoc)) {`.
  **L3518 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::RelaxedCLoc)) {`。
- **L3519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(common::UsageWarning::CLoc, arguments[0]->sourceLocation(),`.
  **L3519 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(common::UsageWarning::CLoc, arguments[0]->sourceLocation(),`。
- **L3520 EN**: Executes a call or declaration centered on `"C_LOC`.
  **L3520 CN**: 执行以 `"C_LOC` 为核心的调用或声明。
- **L3521 EN**: Transitions from the previous branch into the alternative path.
  **L3521 CN**: 从前一个分支过渡到备选路径。
- **L3522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3522 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3523 EN**: Executes a call or declaration centered on `"C_LOC`.
  **L3523 CN**: 执行以 `"C_LOC` 为核心的调用或声明。
- **L3524 EN**: Closes the current lexical scope or compound statement.
  **L3524 CN**: 结束当前词法作用域或复合语句块。
- **L3525 EN**: Closes the current lexical scope or compound statement.
  **L3525 CN**: 结束当前词法作用域或复合语句块。
- **L3526 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3526 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3527 EN**: Continues the surrounding expression or declaration: `arguments[0], context)}) {`.
  **L3527 CN**: 继续构造周围的表达式或声明：`arguments[0], context)}) {`。
- **L3528 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3528 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3529-3552

````cpp
        context.messages().Say(arguments[0]->sourceLocation(),
            "C_LOC() argument must be contiguous"_err_en_US);
      }
      if (auto constExtents{AsConstantExtents(context, typeAndShape->shape())};
          constExtents && GetSize(*constExtents) == 0) {
        context.messages().Say(arguments[0]->sourceLocation(),
            "C_LOC() argument may not be a zero-sized array"_err_en_US);
      }
      if (!(typeAndShape->type().category() != TypeCategory::Derived ||
              typeAndShape->type().IsAssumedType() ||
              (!typeAndShape->type().IsPolymorphic() &&
                  CountNonConstantLenParameters(
                      typeAndShape->type().GetDerivedTypeSpec()) == 0))) {
        context.messages().Say(arguments[0]->sourceLocation(),
            "C_LOC() argument must have an intrinsic type, assumed type, or non-polymorphic derived type with no non-constant length parameter"_err_en_US);
      } else if (typeAndShape->type().knownLength().value_or(1) == 0) {
        context.messages().Say(arguments[0]->sourceLocation(),
            "C_LOC() argument may not be zero-length character"_err_en_US);
      } else if (typeAndShape->type().category() != TypeCategory::Derived &&
          !IsInteroperableIntrinsicType(typeAndShape->type()).value_or(true)) {
        if (typeAndShape->type().category() == TypeCategory::Character &&
            typeAndShape->type().kind() == 1) {
          // Default character kind, but length is not known to be 1
          context.Warn(common::UsageWarning::CharacterInteroperability,
````
- **L3529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3529 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3530 EN**: Executes a call or declaration centered on `"C_LOC`.
  **L3530 CN**: 执行以 `"C_LOC` 为核心的调用或声明。
- **L3531 EN**: Closes the current lexical scope or compound statement.
  **L3531 CN**: 结束当前词法作用域或复合语句块。
- **L3532 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3532 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3533 EN**: Starts a function, method, lambda, or structured scope: `constExtents && GetSize(*constExtents) == 0) {`.
  **L3533 CN**: 开始一个函数、方法、lambda 或结构化作用域：`constExtents && GetSize(*constExtents) == 0) {`。
- **L3534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3534 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3535 EN**: Executes a call or declaration centered on `"C_LOC`.
  **L3535 CN**: 执行以 `"C_LOC` 为核心的调用或声明。
- **L3536 EN**: Closes the current lexical scope or compound statement.
  **L3536 CN**: 结束当前词法作用域或复合语句块。
- **L3537 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3537 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3538 EN**: Continues logic associated with callable symbol `type`.
  **L3538 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L3539 EN**: Continues logic associated with callable symbol `type`.
  **L3539 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L3540 EN**: Continues logic associated with callable symbol `CountNonConstantLenParameters`.
  **L3540 CN**: 继续与可调用符号 `CountNonConstantLenParameters` 相关的逻辑。
- **L3541 EN**: Starts a function, method, lambda, or structured scope: `typeAndShape->type().GetDerivedTypeSpec()) == 0))) {`.
  **L3541 CN**: 开始一个函数、方法、lambda 或结构化作用域：`typeAndShape->type().GetDerivedTypeSpec()) == 0))) {`。
- **L3542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3542 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3543 EN**: Executes a call or declaration centered on `"C_LOC`.
  **L3543 CN**: 执行以 `"C_LOC` 为核心的调用或声明。
- **L3544 EN**: Transitions from the previous branch into an `else if` condition.
  **L3544 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3545 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3546 EN**: Executes a call or declaration centered on `"C_LOC`.
  **L3546 CN**: 执行以 `"C_LOC` 为核心的调用或声明。
- **L3547 EN**: Transitions from the previous branch into an `else if` condition.
  **L3547 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3548 EN**: Starts a function, method, lambda, or structured scope: `!IsInteroperableIntrinsicType(typeAndShape->type()).value_or(true)) {`.
  **L3548 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!IsInteroperableIntrinsicType(typeAndShape->type()).value_or(true)) {`。
- **L3549 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3549 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3550 EN**: Starts a function, method, lambda, or structured scope: `typeAndShape->type().kind() == 1) {`.
  **L3550 CN**: 开始一个函数、方法、lambda 或结构化作用域：`typeAndShape->type().kind() == 1) {`。
- **L3551 EN**: Comment explains nearby logic, intent, or metadata: `Default character kind, but length is not known to be 1`.
  **L3551 CN**: 注释说明附近代码的逻辑、意图或元数据：`Default character kind, but length is not known to be 1`。
- **L3552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(common::UsageWarning::CharacterInteroperability,`.
  **L3552 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(common::UsageWarning::CharacterInteroperability,`。

### Lines 3553-3576

````cpp
              arguments[0]->sourceLocation(),
              "C_LOC() argument has non-interoperable character length"_warn_en_US);
        } else {
          context.Warn(common::UsageWarning::Interoperability,
              arguments[0]->sourceLocation(),
              "C_LOC() argument has non-interoperable intrinsic type or kind"_warn_en_US);
        }
      }
      characteristics::DummyDataObject ddo{std::move(*typeAndShape)};
      ddo.intent = common::Intent::In;
      specificCall.specificIntrinsic.characteristics.value()
          .dummyArguments.emplace_back(
              characteristics::DummyArgument{"x", std::move(ddo)});
      specificCall.arguments.emplace_back(std::move(arguments[0]));
      return specificCall;
    } else if (context.languageFeatures().IsEnabled(
                   common::LanguageFeature::RelaxedCLoc)) {
      if (!expr || !IsProcedurePointer(*expr)) {
        // There are more specific errors as to why the expression doesn't exist
        // or isn't characterizable as a data object or procedure.
      } else if (auto proc{characteristics::Procedure::Characterize(
                     *expr, context)}) {
        characteristics::DummyProcedure dProc{std::move(*proc)};
        dProc.intent = common::Intent::In;
````
- **L3553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arguments[0]->sourceLocation(),`.
  **L3553 CN**: 继续一个多行参数列表、初始化器或聚合项：`arguments[0]->sourceLocation(),`。
- **L3554 EN**: Executes a call or declaration centered on `"C_LOC`.
  **L3554 CN**: 执行以 `"C_LOC` 为核心的调用或声明。
- **L3555 EN**: Transitions from the previous branch into the alternative path.
  **L3555 CN**: 从前一个分支过渡到备选路径。
- **L3556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(common::UsageWarning::Interoperability,`.
  **L3556 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(common::UsageWarning::Interoperability,`。
- **L3557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arguments[0]->sourceLocation(),`.
  **L3557 CN**: 继续一个多行参数列表、初始化器或聚合项：`arguments[0]->sourceLocation(),`。
- **L3558 EN**: Executes a call or declaration centered on `"C_LOC`.
  **L3558 CN**: 执行以 `"C_LOC` 为核心的调用或声明。
- **L3559 EN**: Closes the current lexical scope or compound statement.
  **L3559 CN**: 结束当前词法作用域或复合语句块。
- **L3560 EN**: Closes the current lexical scope or compound statement.
  **L3560 CN**: 结束当前词法作用域或复合语句块。
- **L3561 EN**: Executes a call or declaration centered on `ddo{std::move`.
  **L3561 CN**: 执行以 `ddo{std::move` 为核心的调用或声明。
- **L3562 EN**: Executes a standalone statement or declaration: `ddo.intent = common::Intent::In;`.
  **L3562 CN**: 执行一条独立语句或声明：`ddo.intent = common::Intent::In;`。
- **L3563 EN**: Continues logic associated with callable symbol `value`.
  **L3563 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L3564 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L3564 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L3565 EN**: Executes a call or declaration centered on `std::move`.
  **L3565 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3566 EN**: Executes a call or declaration centered on `specificCall.arguments.emplace_back`.
  **L3566 CN**: 执行以 `specificCall.arguments.emplace_back` 为核心的调用或声明。
- **L3567 EN**: Returns from the current function with `specificCall`.
  **L3567 CN**: 以 `specificCall` 从当前函数返回。
- **L3568 EN**: Transitions from the previous branch into an `else if` condition.
  **L3568 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3569 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::RelaxedCLoc)) {`.
  **L3569 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::RelaxedCLoc)) {`。
- **L3570 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3570 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3571 EN**: Comment explains nearby logic, intent, or metadata: `There are more specific errors as to why the expression doesn't exist`.
  **L3571 CN**: 注释说明附近代码的逻辑、意图或元数据：`There are more specific errors as to why the expression doesn't exist`。
- **L3572 EN**: Comment explains nearby logic, intent, or metadata: `or isn't characterizable as a data object or procedure.`.
  **L3572 CN**: 注释说明附近代码的逻辑、意图或元数据：`or isn't characterizable as a data object or procedure.`。
- **L3573 EN**: Transitions from the previous branch into an `else if` condition.
  **L3573 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3574 EN**: Comment explains nearby logic, intent, or metadata: `expr, context)}) {`.
  **L3574 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr, context)}) {`。
- **L3575 EN**: Executes a call or declaration centered on `dProc{std::move`.
  **L3575 CN**: 执行以 `dProc{std::move` 为核心的调用或声明。
- **L3576 EN**: Executes a standalone statement or declaration: `dProc.intent = common::Intent::In;`.
  **L3576 CN**: 执行一条独立语句或声明：`dProc.intent = common::Intent::In;`。

### Lines 3577-3600

````cpp
        specificCall.specificIntrinsic.characteristics.value()
            .dummyArguments.emplace_back(
                characteristics::DummyArgument{"x", std::move(dProc)});
        specificCall.arguments.emplace_back(std::move(arguments[0]));
        return specificCall;
      }
    }
  }
  return std::nullopt;
}

// CUDA Fortran C_DEVLOC(x)
std::optional<SpecificCall> IntrinsicProcTable::Implementation::HandleC_Devloc(
    ActualArguments &arguments, FoldingContext &context) const {
  static const char *const keywords[]{"cptr", nullptr};

  if (CheckAndRearrangeArguments(arguments, context.messages(), keywords)) {
    CHECK(arguments.size() == 1);
    const auto *expr{arguments[0].value().UnwrapExpr()};
    if (auto typeAndShape{characteristics::TypeAndShape::Characterize(
            arguments[0], context)}) {
      if (expr && !IsContiguous(*expr, context).value_or(true)) {
        context.messages().Say(arguments[0]->sourceLocation(),
            "C_DEVLOC() argument must be contiguous"_err_en_US);
````
- **L3577 EN**: Continues logic associated with callable symbol `value`.
  **L3577 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L3578 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L3578 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L3579 EN**: Executes a call or declaration centered on `std::move`.
  **L3579 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3580 EN**: Executes a call or declaration centered on `specificCall.arguments.emplace_back`.
  **L3580 CN**: 执行以 `specificCall.arguments.emplace_back` 为核心的调用或声明。
- **L3581 EN**: Returns from the current function with `specificCall`.
  **L3581 CN**: 以 `specificCall` 从当前函数返回。
- **L3582 EN**: Closes the current lexical scope or compound statement.
  **L3582 CN**: 结束当前词法作用域或复合语句块。
- **L3583 EN**: Closes the current lexical scope or compound statement.
  **L3583 CN**: 结束当前词法作用域或复合语句块。
- **L3584 EN**: Closes the current lexical scope or compound statement.
  **L3584 CN**: 结束当前词法作用域或复合语句块。
- **L3585 EN**: Returns from the current function with `std::nullopt`.
  **L3585 CN**: 以 `std::nullopt` 从当前函数返回。
- **L3586 EN**: Closes the current lexical scope or compound statement.
  **L3586 CN**: 结束当前词法作用域或复合语句块。
- **L3587 EN**: Blank line separating nearby declarations or logic blocks.
  **L3587 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3588 EN**: Comment explains nearby logic, intent, or metadata: `CUDA Fortran C_DEVLOC(x)`.
  **L3588 CN**: 注释说明附近代码的逻辑、意图或元数据：`CUDA Fortran C_DEVLOC(x)`。
- **L3589 EN**: Continues logic associated with callable symbol `HandleC_Devloc`.
  **L3589 CN**: 继续与可调用符号 `HandleC_Devloc` 相关的逻辑。
- **L3590 EN**: Continues the surrounding expression or declaration: `ActualArguments &arguments, FoldingContext &context) const {`.
  **L3590 CN**: 继续构造周围的表达式或声明：`ActualArguments &arguments, FoldingContext &context) const {`。
- **L3591 EN**: Executes a standalone statement or declaration: `static const char *const keywords[]{"cptr", nullptr};`.
  **L3591 CN**: 执行一条独立语句或声明：`static const char *const keywords[]{"cptr", nullptr};`。
- **L3592 EN**: Blank line separating nearby declarations or logic blocks.
  **L3592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3593 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3593 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3594 EN**: Executes a call or declaration centered on `CHECK`.
  **L3594 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L3595 EN**: Executes a call or declaration centered on `*expr{arguments[0].value`.
  **L3595 CN**: 执行以 `*expr{arguments[0].value` 为核心的调用或声明。
- **L3596 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3596 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3597 EN**: Continues the surrounding expression or declaration: `arguments[0], context)}) {`.
  **L3597 CN**: 继续构造周围的表达式或声明：`arguments[0], context)}) {`。
- **L3598 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3598 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3599 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3600 EN**: Executes a call or declaration centered on `"C_DEVLOC`.
  **L3600 CN**: 执行以 `"C_DEVLOC` 为核心的调用或声明。

### Lines 3601-3624

````cpp
      }
      if (auto constExtents{AsConstantExtents(context, typeAndShape->shape())};
          constExtents && GetSize(*constExtents) == 0) {
        context.messages().Say(arguments[0]->sourceLocation(),
            "C_DEVLOC() argument may not be a zero-sized array"_err_en_US);
      }
      if (!(typeAndShape->type().category() != TypeCategory::Derived ||
              typeAndShape->type().IsAssumedType() ||
              (!typeAndShape->type().IsPolymorphic() &&
                  CountNonConstantLenParameters(
                      typeAndShape->type().GetDerivedTypeSpec()) == 0))) {
        context.messages().Say(arguments[0]->sourceLocation(),
            "C_DEVLOC() argument must have an intrinsic type, assumed type, or non-polymorphic derived type with no non-constant length parameter"_err_en_US);
      } else if (typeAndShape->type().knownLength().value_or(1) == 0) {
        context.messages().Say(arguments[0]->sourceLocation(),
            "C_DEVLOC() argument may not be zero-length character"_err_en_US);
      } else if (typeAndShape->type().category() != TypeCategory::Derived &&
          !IsInteroperableIntrinsicType(typeAndShape->type()).value_or(true)) {
        if (typeAndShape->type().category() == TypeCategory::Character &&
            typeAndShape->type().kind() == 1) {
          // Default character kind, but length is not known to be 1
          context.Warn(common::UsageWarning::CharacterInteroperability,
              arguments[0]->sourceLocation(),
              "C_DEVLOC() argument has non-interoperable character length"_warn_en_US);
````
- **L3601 EN**: Closes the current lexical scope or compound statement.
  **L3601 CN**: 结束当前词法作用域或复合语句块。
- **L3602 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3602 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3603 EN**: Starts a function, method, lambda, or structured scope: `constExtents && GetSize(*constExtents) == 0) {`.
  **L3603 CN**: 开始一个函数、方法、lambda 或结构化作用域：`constExtents && GetSize(*constExtents) == 0) {`。
- **L3604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3604 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3605 EN**: Executes a call or declaration centered on `"C_DEVLOC`.
  **L3605 CN**: 执行以 `"C_DEVLOC` 为核心的调用或声明。
- **L3606 EN**: Closes the current lexical scope or compound statement.
  **L3606 CN**: 结束当前词法作用域或复合语句块。
- **L3607 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3607 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3608 EN**: Continues logic associated with callable symbol `type`.
  **L3608 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L3609 EN**: Continues logic associated with callable symbol `type`.
  **L3609 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L3610 EN**: Continues logic associated with callable symbol `CountNonConstantLenParameters`.
  **L3610 CN**: 继续与可调用符号 `CountNonConstantLenParameters` 相关的逻辑。
- **L3611 EN**: Starts a function, method, lambda, or structured scope: `typeAndShape->type().GetDerivedTypeSpec()) == 0))) {`.
  **L3611 CN**: 开始一个函数、方法、lambda 或结构化作用域：`typeAndShape->type().GetDerivedTypeSpec()) == 0))) {`。
- **L3612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3612 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3613 EN**: Executes a call or declaration centered on `"C_DEVLOC`.
  **L3613 CN**: 执行以 `"C_DEVLOC` 为核心的调用或声明。
- **L3614 EN**: Transitions from the previous branch into an `else if` condition.
  **L3614 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arguments[0]->sourceLocation(),`.
  **L3615 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arguments[0]->sourceLocation(),`。
- **L3616 EN**: Executes a call or declaration centered on `"C_DEVLOC`.
  **L3616 CN**: 执行以 `"C_DEVLOC` 为核心的调用或声明。
- **L3617 EN**: Transitions from the previous branch into an `else if` condition.
  **L3617 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3618 EN**: Starts a function, method, lambda, or structured scope: `!IsInteroperableIntrinsicType(typeAndShape->type()).value_or(true)) {`.
  **L3618 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!IsInteroperableIntrinsicType(typeAndShape->type()).value_or(true)) {`。
- **L3619 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3619 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3620 EN**: Starts a function, method, lambda, or structured scope: `typeAndShape->type().kind() == 1) {`.
  **L3620 CN**: 开始一个函数、方法、lambda 或结构化作用域：`typeAndShape->type().kind() == 1) {`。
- **L3621 EN**: Comment explains nearby logic, intent, or metadata: `Default character kind, but length is not known to be 1`.
  **L3621 CN**: 注释说明附近代码的逻辑、意图或元数据：`Default character kind, but length is not known to be 1`。
- **L3622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(common::UsageWarning::CharacterInteroperability,`.
  **L3622 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(common::UsageWarning::CharacterInteroperability,`。
- **L3623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arguments[0]->sourceLocation(),`.
  **L3623 CN**: 继续一个多行参数列表、初始化器或聚合项：`arguments[0]->sourceLocation(),`。
- **L3624 EN**: Executes a call or declaration centered on `"C_DEVLOC`.
  **L3624 CN**: 执行以 `"C_DEVLOC` 为核心的调用或声明。

### Lines 3625-3648

````cpp
        } else {
          context.Warn(common::UsageWarning::Interoperability,
              arguments[0]->sourceLocation(),
              "C_DEVLOC() argument has non-interoperable intrinsic type or kind"_warn_en_US);
        }
      }

      characteristics::DummyDataObject ddo{std::move(*typeAndShape)};
      ddo.intent = common::Intent::In;
      return SpecificCall{
          SpecificIntrinsic{"__builtin_c_devloc"s,
              characteristics::Procedure{
                  characteristics::FunctionResult{
                      DynamicType{GetBuiltinDerivedType(
                          builtinsScope_, "__builtin_c_devptr")}},
                  characteristics::DummyArguments{
                      characteristics::DummyArgument{"cptr"s, std::move(ddo)}},
                  characteristics::Procedure::Attrs{
                      characteristics::Procedure::Attr::Pure}}},
          std::move(arguments)};
    }
  }
  return std::nullopt;
}
````
- **L3625 EN**: Transitions from the previous branch into the alternative path.
  **L3625 CN**: 从前一个分支过渡到备选路径。
- **L3626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(common::UsageWarning::Interoperability,`.
  **L3626 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(common::UsageWarning::Interoperability,`。
- **L3627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arguments[0]->sourceLocation(),`.
  **L3627 CN**: 继续一个多行参数列表、初始化器或聚合项：`arguments[0]->sourceLocation(),`。
- **L3628 EN**: Executes a call or declaration centered on `"C_DEVLOC`.
  **L3628 CN**: 执行以 `"C_DEVLOC` 为核心的调用或声明。
- **L3629 EN**: Closes the current lexical scope or compound statement.
  **L3629 CN**: 结束当前词法作用域或复合语句块。
- **L3630 EN**: Closes the current lexical scope or compound statement.
  **L3630 CN**: 结束当前词法作用域或复合语句块。
- **L3631 EN**: Blank line separating nearby declarations or logic blocks.
  **L3631 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3632 EN**: Executes a call or declaration centered on `ddo{std::move`.
  **L3632 CN**: 执行以 `ddo{std::move` 为核心的调用或声明。
- **L3633 EN**: Executes a standalone statement or declaration: `ddo.intent = common::Intent::In;`.
  **L3633 CN**: 执行一条独立语句或声明：`ddo.intent = common::Intent::In;`。
- **L3634 EN**: Returns from the current function with `SpecificCall{`.
  **L3634 CN**: 以 `SpecificCall{` 从当前函数返回。
- **L3635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SpecificIntrinsic{"__builtin_c_devloc"s,`.
  **L3635 CN**: 继续一个多行参数列表、初始化器或聚合项：`SpecificIntrinsic{"__builtin_c_devloc"s,`。
- **L3636 EN**: Continues the surrounding expression or declaration: `characteristics::Procedure{`.
  **L3636 CN**: 继续构造周围的表达式或声明：`characteristics::Procedure{`。
- **L3637 EN**: Continues the surrounding expression or declaration: `characteristics::FunctionResult{`.
  **L3637 CN**: 继续构造周围的表达式或声明：`characteristics::FunctionResult{`。
- **L3638 EN**: Continues logic associated with callable symbol `GetBuiltinDerivedType`.
  **L3638 CN**: 继续与可调用符号 `GetBuiltinDerivedType` 相关的逻辑。
- **L3639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builtinsScope_, "__builtin_c_devptr")}},`.
  **L3639 CN**: 继续一个多行参数列表、初始化器或聚合项：`builtinsScope_, "__builtin_c_devptr")}},`。
- **L3640 EN**: Continues the surrounding expression or declaration: `characteristics::DummyArguments{`.
  **L3640 CN**: 继续构造周围的表达式或声明：`characteristics::DummyArguments{`。
- **L3641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyArgument{"cptr"s, std::move(ddo)}},`.
  **L3641 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyArgument{"cptr"s, std::move(ddo)}},`。
- **L3642 EN**: Continues the surrounding expression or declaration: `characteristics::Procedure::Attrs{`.
  **L3642 CN**: 继续构造周围的表达式或声明：`characteristics::Procedure::Attrs{`。
- **L3643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::Procedure::Attr::Pure}}},`.
  **L3643 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::Procedure::Attr::Pure}}},`。
- **L3644 EN**: Executes a call or declaration centered on `std::move`.
  **L3644 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3645 EN**: Closes the current lexical scope or compound statement.
  **L3645 CN**: 结束当前词法作用域或复合语句块。
- **L3646 EN**: Closes the current lexical scope or compound statement.
  **L3646 CN**: 结束当前词法作用域或复合语句块。
- **L3647 EN**: Returns from the current function with `std::nullopt`.
  **L3647 CN**: 以 `std::nullopt` 从当前函数返回。
- **L3648 EN**: Closes the current lexical scope or compound statement.
  **L3648 CN**: 结束当前词法作用域或复合语句块。

### Lines 3649-3672

````cpp

static bool CheckForNonPositiveValues(FoldingContext &context,
    const ActualArgument &arg, const std::string &procName,
    const std::string &argName) {
  bool ok{true};
  if (arg.Rank() > 0) {
    if (const Expr<SomeType> *expr{arg.UnwrapExpr()}) {
      if (const auto *intExpr{std::get_if<Expr<SomeInteger>>(&expr->u)}) {
        Fortran::common::visit(
            [&](const auto &kindExpr) {
              using IntType = typename std::decay_t<decltype(kindExpr)>::Result;
              if (const auto *constArray{
                      UnwrapConstantValue<IntType>(kindExpr)}) {
                for (std::size_t j{0}; j < constArray->size(); ++j) {
                  auto arrayExpr{constArray->values().at(j)};
                  if (arrayExpr.IsNegative() || arrayExpr.IsZero()) {
                    ok = false;
                    context.messages().Say(arg.sourceLocation(),
                        "'%s=' argument for intrinsic '%s' must contain all positive values"_err_en_US,
                        argName, procName);
                  }
                }
              }
            },
````
- **L3649 EN**: Blank line separating nearby declarations or logic blocks.
  **L3649 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static bool CheckForNonPositiveValues(FoldingContext &context,`.
  **L3650 CN**: 继续一个多行参数列表、初始化器或聚合项：`static bool CheckForNonPositiveValues(FoldingContext &context,`。
- **L3651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const ActualArgument &arg, const std::string &procName,`.
  **L3651 CN**: 继续一个多行参数列表、初始化器或聚合项：`const ActualArgument &arg, const std::string &procName,`。
- **L3652 EN**: Continues the surrounding expression or declaration: `const std::string &argName) {`.
  **L3652 CN**: 继续构造周围的表达式或声明：`const std::string &argName) {`。
- **L3653 EN**: Executes a standalone statement or declaration: `bool ok{true};`.
  **L3653 CN**: 执行一条独立语句或声明：`bool ok{true};`。
- **L3654 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3654 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3655 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3655 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3656 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3656 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3657 EN**: Continues logic associated with callable symbol `visit`.
  **L3657 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L3658 EN**: Starts a function, method, lambda, or structured scope: `[&](const auto &kindExpr) {`.
  **L3658 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const auto &kindExpr) {`。
- **L3659 EN**: Defines alias `IntType` to simplify later code.
  **L3659 CN**: 定义别名 `IntType` 以简化后续代码。
- **L3660 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3660 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3661 EN**: Starts a function, method, lambda, or structured scope: `UnwrapConstantValue<IntType>(kindExpr)}) {`.
  **L3661 CN**: 开始一个函数、方法、lambda 或结构化作用域：`UnwrapConstantValue<IntType>(kindExpr)}) {`。
- **L3662 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3662 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3663 EN**: Executes a call or declaration centered on `arrayExpr{constArray->values`.
  **L3663 CN**: 执行以 `arrayExpr{constArray->values` 为核心的调用或声明。
- **L3664 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3664 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3665 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L3665 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L3666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arg.sourceLocation(),`.
  **L3666 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arg.sourceLocation(),`。
- **L3667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s=' argument for intrinsic '%s' must contain all positive values"_err_en_US,`.
  **L3667 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s=' argument for intrinsic '%s' must contain all positive values"_err_en_US,`。
- **L3668 EN**: Executes a standalone statement or declaration: `argName, procName);`.
  **L3668 CN**: 执行一条独立语句或声明：`argName, procName);`。
- **L3669 EN**: Closes the current lexical scope or compound statement.
  **L3669 CN**: 结束当前词法作用域或复合语句块。
- **L3670 EN**: Closes the current lexical scope or compound statement.
  **L3670 CN**: 结束当前词法作用域或复合语句块。
- **L3671 EN**: Closes the current lexical scope or compound statement.
  **L3671 CN**: 结束当前词法作用域或复合语句块。
- **L3672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L3672 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 3673-3696

````cpp
            intExpr->u);
      }
    }
  } else {
    if (auto val{ToInt64(arg.UnwrapExpr())}) {
      if (*val <= 0) {
        ok = false;
        context.messages().Say(arg.sourceLocation(),
            "'%s=' argument for intrinsic '%s' must be a positive value, but is %jd"_err_en_US,
            argName, procName, static_cast<std::intmax_t>(*val));
      }
    }
  }
  return ok;
}

static bool CheckAtomicDefineAndRef(FoldingContext &context,
    const std::optional<ActualArgument> &atomArg,
    const std::optional<ActualArgument> &valueArg,
    const std::optional<ActualArgument> &statArg, const std::string &procName) {
  bool sameType{true};
  if (valueArg && atomArg) {
    // for atomic_define and atomic_ref, 'value' arg must be the same type as
    // 'atom', but it doesn't have to be the same kind
````
- **L3673 EN**: Executes a standalone statement or declaration: `intExpr->u);`.
  **L3673 CN**: 执行一条独立语句或声明：`intExpr->u);`。
- **L3674 EN**: Closes the current lexical scope or compound statement.
  **L3674 CN**: 结束当前词法作用域或复合语句块。
- **L3675 EN**: Closes the current lexical scope or compound statement.
  **L3675 CN**: 结束当前词法作用域或复合语句块。
- **L3676 EN**: Transitions from the previous branch into the alternative path.
  **L3676 CN**: 从前一个分支过渡到备选路径。
- **L3677 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3677 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3678 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3678 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3679 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L3679 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L3680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arg.sourceLocation(),`.
  **L3680 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arg.sourceLocation(),`。
- **L3681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s=' argument for intrinsic '%s' must be a positive value, but is %jd"_err_en_US,`.
  **L3681 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s=' argument for intrinsic '%s' must be a positive value, but is %jd"_err_en_US,`。
- **L3682 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L3682 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L3683 EN**: Closes the current lexical scope or compound statement.
  **L3683 CN**: 结束当前词法作用域或复合语句块。
- **L3684 EN**: Closes the current lexical scope or compound statement.
  **L3684 CN**: 结束当前词法作用域或复合语句块。
- **L3685 EN**: Closes the current lexical scope or compound statement.
  **L3685 CN**: 结束当前词法作用域或复合语句块。
- **L3686 EN**: Returns from the current function with `ok`.
  **L3686 CN**: 以 `ok` 从当前函数返回。
- **L3687 EN**: Closes the current lexical scope or compound statement.
  **L3687 CN**: 结束当前词法作用域或复合语句块。
- **L3688 EN**: Blank line separating nearby declarations or logic blocks.
  **L3688 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static bool CheckAtomicDefineAndRef(FoldingContext &context,`.
  **L3689 CN**: 继续一个多行参数列表、初始化器或聚合项：`static bool CheckAtomicDefineAndRef(FoldingContext &context,`。
- **L3690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const std::optional<ActualArgument> &atomArg,`.
  **L3690 CN**: 继续一个多行参数列表、初始化器或聚合项：`const std::optional<ActualArgument> &atomArg,`。
- **L3691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const std::optional<ActualArgument> &valueArg,`.
  **L3691 CN**: 继续一个多行参数列表、初始化器或聚合项：`const std::optional<ActualArgument> &valueArg,`。
- **L3692 EN**: Continues the surrounding expression or declaration: `const std::optional<ActualArgument> &statArg, const std::string &procName) {`.
  **L3692 CN**: 继续构造周围的表达式或声明：`const std::optional<ActualArgument> &statArg, const std::string &procName) {`。
- **L3693 EN**: Executes a standalone statement or declaration: `bool sameType{true};`.
  **L3693 CN**: 执行一条独立语句或声明：`bool sameType{true};`。
- **L3694 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3694 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3695 EN**: Comment explains nearby logic, intent, or metadata: `for atomic_define and atomic_ref, 'value' arg must be the same type as`.
  **L3695 CN**: 注释说明附近代码的逻辑、意图或元数据：`for atomic_define and atomic_ref, 'value' arg must be the same type as`。
- **L3696 EN**: Comment explains nearby logic, intent, or metadata: `'atom', but it doesn't have to be the same kind`.
  **L3696 CN**: 注释说明附近代码的逻辑、意图或元数据：`'atom', but it doesn't have to be the same kind`。

### Lines 3697-3720

````cpp
    if (valueArg->GetType()->category() != atomArg->GetType()->category()) {
      sameType = false;
      context.messages().Say(valueArg->sourceLocation(),
          "'value=' argument to '%s' must have same type as 'atom=', but is '%s'"_err_en_US,
          procName, valueArg->GetType()->AsFortran());
    }
  }

  return sameType &&
      CheckForCoindexedObject(context.messages(), statArg, procName, "stat");
}

// Applies any semantic checks peculiar to an intrinsic.
// TODO: Move the rest of these checks to Semantics/check-call.cpp.
static bool ApplySpecificChecks(SpecificCall &call, FoldingContext &context) {
  bool ok{true};
  const std::string &name{call.specificIntrinsic.name};
  if (name == "allocated") {
    const auto &arg{call.arguments[0]};
    if (arg) {
      if (const auto *expr{arg->UnwrapExpr()}) {
        ok = IsAllocatableDesignator(*expr) || IsNullAllocatable(expr);
      }
    }
````
- **L3697 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3697 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3698 EN**: Executes a standalone statement or declaration: `sameType = false;`.
  **L3698 CN**: 执行一条独立语句或声明：`sameType = false;`。
- **L3699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(valueArg->sourceLocation(),`.
  **L3699 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(valueArg->sourceLocation(),`。
- **L3700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'value=' argument to '%s' must have same type as 'atom=', but is '%s'"_err_en_US,`.
  **L3700 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'value=' argument to '%s' must have same type as 'atom=', but is '%s'"_err_en_US,`。
- **L3701 EN**: Executes a call or declaration centered on `valueArg->GetType`.
  **L3701 CN**: 执行以 `valueArg->GetType` 为核心的调用或声明。
- **L3702 EN**: Closes the current lexical scope or compound statement.
  **L3702 CN**: 结束当前词法作用域或复合语句块。
- **L3703 EN**: Closes the current lexical scope or compound statement.
  **L3703 CN**: 结束当前词法作用域或复合语句块。
- **L3704 EN**: Blank line separating nearby declarations or logic blocks.
  **L3704 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3705 EN**: Returns from the current function with `sameType &&`.
  **L3705 CN**: 以 `sameType &&` 从当前函数返回。
- **L3706 EN**: Executes a call or declaration centered on `CheckForCoindexedObject`.
  **L3706 CN**: 执行以 `CheckForCoindexedObject` 为核心的调用或声明。
- **L3707 EN**: Closes the current lexical scope or compound statement.
  **L3707 CN**: 结束当前词法作用域或复合语句块。
- **L3708 EN**: Blank line separating nearby declarations or logic blocks.
  **L3708 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3709 EN**: Comment explains nearby logic, intent, or metadata: `Applies any semantic checks peculiar to an intrinsic.`.
  **L3709 CN**: 注释说明附近代码的逻辑、意图或元数据：`Applies any semantic checks peculiar to an intrinsic.`。
- **L3710 EN**: Comment records a pending task or caution: `TODO: Move the rest of these checks to Semantics/check-call.cpp.`.
  **L3710 CN**: 注释记录待办事项或注意点：`TODO: Move the rest of these checks to Semantics/check-call.cpp.`。
- **L3711 EN**: Starts a function, method, lambda, or structured scope: `static bool ApplySpecificChecks(SpecificCall &call, FoldingContext &context) {`.
  **L3711 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool ApplySpecificChecks(SpecificCall &call, FoldingContext &context) {`。
- **L3712 EN**: Executes a standalone statement or declaration: `bool ok{true};`.
  **L3712 CN**: 执行一条独立语句或声明：`bool ok{true};`。
- **L3713 EN**: Executes a standalone statement or declaration: `const std::string &name{call.specificIntrinsic.name};`.
  **L3713 CN**: 执行一条独立语句或声明：`const std::string &name{call.specificIntrinsic.name};`。
- **L3714 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3714 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3715 EN**: Executes a standalone statement or declaration: `const auto &arg{call.arguments[0]};`.
  **L3715 CN**: 执行一条独立语句或声明：`const auto &arg{call.arguments[0]};`。
- **L3716 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3716 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3717 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3717 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3718 EN**: Executes a call or declaration centered on `IsAllocatableDesignator`.
  **L3718 CN**: 执行以 `IsAllocatableDesignator` 为核心的调用或声明。
- **L3719 EN**: Closes the current lexical scope or compound statement.
  **L3719 CN**: 结束当前词法作用域或复合语句块。
- **L3720 EN**: Closes the current lexical scope or compound statement.
  **L3720 CN**: 结束当前词法作用域或复合语句块。

### Lines 3721-3744

````cpp
    if (!ok) {
      context.messages().Say(
          arg ? arg->sourceLocation() : context.messages().at(),
          "Argument of ALLOCATED() must be an ALLOCATABLE object or component"_err_en_US);
    }
  } else if (name == "atomic_add" || name == "atomic_and" ||
      name == "atomic_or" || name == "atomic_xor" || name == "event_query") {
    return CheckForCoindexedObject(
        context.messages(), call.arguments[2], name, "stat");
  } else if (name == "atomic_cas") {
    return CheckForCoindexedObject(
        context.messages(), call.arguments[4], name, "stat");
  } else if (name == "atomic_define") {
    return CheckAtomicDefineAndRef(
        context, call.arguments[0], call.arguments[1], call.arguments[2], name);
  } else if (name == "atomic_fetch_add" || name == "atomic_fetch_and" ||
      name == "atomic_fetch_or" || name == "atomic_fetch_xor") {
    return CheckForCoindexedObject(
        context.messages(), call.arguments[3], name, "stat");
  } else if (name == "atomic_ref") {
    return CheckAtomicDefineAndRef(
        context, call.arguments[1], call.arguments[0], call.arguments[2], name);
  } else if (name == "co_broadcast" || name == "co_max" || name == "co_min" ||
      name == "co_sum") {
````
- **L3721 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3721 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3722 EN**: Continues logic associated with callable symbol `messages`.
  **L3722 CN**: 继续与可调用符号 `messages` 相关的逻辑。
- **L3723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arg ? arg->sourceLocation() : context.messages().at(),`.
  **L3723 CN**: 继续一个多行参数列表、初始化器或聚合项：`arg ? arg->sourceLocation() : context.messages().at(),`。
- **L3724 EN**: Executes a call or declaration centered on `ALLOCATED`.
  **L3724 CN**: 执行以 `ALLOCATED` 为核心的调用或声明。
- **L3725 EN**: Closes the current lexical scope or compound statement.
  **L3725 CN**: 结束当前词法作用域或复合语句块。
- **L3726 EN**: Transitions from the previous branch into an `else if` condition.
  **L3726 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3727 EN**: Continues the surrounding expression or declaration: `name == "atomic_or" || name == "atomic_xor" || name == "event_query") {`.
  **L3727 CN**: 继续构造周围的表达式或声明：`name == "atomic_or" || name == "atomic_xor" || name == "event_query") {`。
- **L3728 EN**: Returns from the current function with `CheckForCoindexedObject(`.
  **L3728 CN**: 以 `CheckForCoindexedObject(` 从当前函数返回。
- **L3729 EN**: Executes a call or declaration centered on `context.messages`.
  **L3729 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L3730 EN**: Transitions from the previous branch into an `else if` condition.
  **L3730 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3731 EN**: Returns from the current function with `CheckForCoindexedObject(`.
  **L3731 CN**: 以 `CheckForCoindexedObject(` 从当前函数返回。
- **L3732 EN**: Executes a call or declaration centered on `context.messages`.
  **L3732 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L3733 EN**: Transitions from the previous branch into an `else if` condition.
  **L3733 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3734 EN**: Returns from the current function with `CheckAtomicDefineAndRef(`.
  **L3734 CN**: 以 `CheckAtomicDefineAndRef(` 从当前函数返回。
- **L3735 EN**: Executes a standalone statement or declaration: `context, call.arguments[0], call.arguments[1], call.arguments[2], name);`.
  **L3735 CN**: 执行一条独立语句或声明：`context, call.arguments[0], call.arguments[1], call.arguments[2], name);`。
- **L3736 EN**: Transitions from the previous branch into an `else if` condition.
  **L3736 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3737 EN**: Continues the surrounding expression or declaration: `name == "atomic_fetch_or" || name == "atomic_fetch_xor") {`.
  **L3737 CN**: 继续构造周围的表达式或声明：`name == "atomic_fetch_or" || name == "atomic_fetch_xor") {`。
- **L3738 EN**: Returns from the current function with `CheckForCoindexedObject(`.
  **L3738 CN**: 以 `CheckForCoindexedObject(` 从当前函数返回。
- **L3739 EN**: Executes a call or declaration centered on `context.messages`.
  **L3739 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L3740 EN**: Transitions from the previous branch into an `else if` condition.
  **L3740 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3741 EN**: Returns from the current function with `CheckAtomicDefineAndRef(`.
  **L3741 CN**: 以 `CheckAtomicDefineAndRef(` 从当前函数返回。
- **L3742 EN**: Executes a standalone statement or declaration: `context, call.arguments[1], call.arguments[0], call.arguments[2], name);`.
  **L3742 CN**: 执行一条独立语句或声明：`context, call.arguments[1], call.arguments[0], call.arguments[2], name);`。
- **L3743 EN**: Transitions from the previous branch into an `else if` condition.
  **L3743 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3744 EN**: Continues the surrounding expression or declaration: `name == "co_sum") {`.
  **L3744 CN**: 继续构造周围的表达式或声明：`name == "co_sum") {`。

### Lines 3745-3768

````cpp
    bool aOk{CheckForCoindexedObject(
        context.messages(), call.arguments[0], name, "a")};
    bool statOk{CheckForCoindexedObject(
        context.messages(), call.arguments[2], name, "stat")};
    bool errmsgOk{CheckForCoindexedObject(
        context.messages(), call.arguments[3], name, "errmsg")};
    ok = aOk && statOk && errmsgOk;
  } else if (name == "image_status") {
    if (const auto &arg{call.arguments[0]}) {
      ok = CheckForNonPositiveValues(context, *arg, name, "image");
    }
  } else if (name == "loc") {
    const auto &arg{call.arguments[0]};
    ok =
        arg && (arg->GetAssumedTypeDummy() || GetLastSymbol(arg->UnwrapExpr()));
    if (!ok) {
      context.messages().Say(
          arg ? arg->sourceLocation() : context.messages().at(),
          "Argument of LOC() must be an object or procedure"_err_en_US);
    }
  } else if (name == "tokenize") {
    // Both forms of TOKENIZE have at least 4 dummy arguments, and the last two
    // must be allocatable.
    const auto &dummies{
````
- **L3745 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L3745 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L3746 EN**: Executes a call or declaration centered on `context.messages`.
  **L3746 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L3747 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L3747 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L3748 EN**: Executes a call or declaration centered on `context.messages`.
  **L3748 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L3749 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L3749 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L3750 EN**: Executes a call or declaration centered on `context.messages`.
  **L3750 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L3751 EN**: Executes a standalone statement or declaration: `ok = aOk && statOk && errmsgOk;`.
  **L3751 CN**: 执行一条独立语句或声明：`ok = aOk && statOk && errmsgOk;`。
- **L3752 EN**: Transitions from the previous branch into an `else if` condition.
  **L3752 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3753 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3753 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3754 EN**: Executes a call or declaration centered on `CheckForNonPositiveValues`.
  **L3754 CN**: 执行以 `CheckForNonPositiveValues` 为核心的调用或声明。
- **L3755 EN**: Closes the current lexical scope or compound statement.
  **L3755 CN**: 结束当前词法作用域或复合语句块。
- **L3756 EN**: Transitions from the previous branch into an `else if` condition.
  **L3756 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3757 EN**: Executes a standalone statement or declaration: `const auto &arg{call.arguments[0]};`.
  **L3757 CN**: 执行一条独立语句或声明：`const auto &arg{call.arguments[0]};`。
- **L3758 EN**: Continues the surrounding expression or declaration: `ok =`.
  **L3758 CN**: 继续构造周围的表达式或声明：`ok =`。
- **L3759 EN**: Executes a call or declaration centered on `&&`.
  **L3759 CN**: 执行以 `&&` 为核心的调用或声明。
- **L3760 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3760 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3761 EN**: Continues logic associated with callable symbol `messages`.
  **L3761 CN**: 继续与可调用符号 `messages` 相关的逻辑。
- **L3762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arg ? arg->sourceLocation() : context.messages().at(),`.
  **L3762 CN**: 继续一个多行参数列表、初始化器或聚合项：`arg ? arg->sourceLocation() : context.messages().at(),`。
- **L3763 EN**: Executes a call or declaration centered on `LOC`.
  **L3763 CN**: 执行以 `LOC` 为核心的调用或声明。
- **L3764 EN**: Closes the current lexical scope or compound statement.
  **L3764 CN**: 结束当前词法作用域或复合语句块。
- **L3765 EN**: Transitions from the previous branch into an `else if` condition.
  **L3765 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3766 EN**: Comment explains nearby logic, intent, or metadata: `Both forms of TOKENIZE have at least 4 dummy arguments, and the last two`.
  **L3766 CN**: 注释说明附近代码的逻辑、意图或元数据：`Both forms of TOKENIZE have at least 4 dummy arguments, and the last two`。
- **L3767 EN**: Comment explains nearby logic, intent, or metadata: `must be allocatable.`.
  **L3767 CN**: 注释说明附近代码的逻辑、意图或元数据：`must be allocatable.`。
- **L3768 EN**: Continues the surrounding expression or declaration: `const auto &dummies{`.
  **L3768 CN**: 继续构造周围的表达式或声明：`const auto &dummies{`。

### Lines 3769-3792

````cpp
        call.specificIntrinsic.characteristics.value().dummyArguments};
    for (int i{2}; i < 4; ++i) {
      const auto &arg{call.arguments[i]};
      if (arg) {
        if (const auto *expr{arg->UnwrapExpr()}) {
          if (!IsAllocatableDesignator(*expr)) {
            ok = false;
            context.messages().Say(arg->sourceLocation(),
                "'%s=' argument to 'tokenize' must be ALLOCATABLE"_err_en_US,
                dummies[i].name);
          }
        }
      }
    }
  }
  return ok;
}

static DynamicType GetReturnType(const SpecificIntrinsicInterface &interface,
    const common::IntrinsicTypeDefaultKinds &defaults) {
  TypeCategory category{TypeCategory::Integer};
  switch (interface.result.kindCode) {
  case KindCode::defaultIntegerKind:
    break;
````
- **L3769 EN**: Executes a call or declaration centered on `call.specificIntrinsic.characteristics.value`.
  **L3769 CN**: 执行以 `call.specificIntrinsic.characteristics.value` 为核心的调用或声明。
- **L3770 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3770 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3771 EN**: Executes a standalone statement or declaration: `const auto &arg{call.arguments[i]};`.
  **L3771 CN**: 执行一条独立语句或声明：`const auto &arg{call.arguments[i]};`。
- **L3772 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3772 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3773 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3773 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3774 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3774 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3775 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L3775 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L3776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.messages().Say(arg->sourceLocation(),`.
  **L3776 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.messages().Say(arg->sourceLocation(),`。
- **L3777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s=' argument to 'tokenize' must be ALLOCATABLE"_err_en_US,`.
  **L3777 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s=' argument to 'tokenize' must be ALLOCATABLE"_err_en_US,`。
- **L3778 EN**: Executes a standalone statement or declaration: `dummies[i].name);`.
  **L3778 CN**: 执行一条独立语句或声明：`dummies[i].name);`。
- **L3779 EN**: Closes the current lexical scope or compound statement.
  **L3779 CN**: 结束当前词法作用域或复合语句块。
- **L3780 EN**: Closes the current lexical scope or compound statement.
  **L3780 CN**: 结束当前词法作用域或复合语句块。
- **L3781 EN**: Closes the current lexical scope or compound statement.
  **L3781 CN**: 结束当前词法作用域或复合语句块。
- **L3782 EN**: Closes the current lexical scope or compound statement.
  **L3782 CN**: 结束当前词法作用域或复合语句块。
- **L3783 EN**: Closes the current lexical scope or compound statement.
  **L3783 CN**: 结束当前词法作用域或复合语句块。
- **L3784 EN**: Returns from the current function with `ok`.
  **L3784 CN**: 以 `ok` 从当前函数返回。
- **L3785 EN**: Closes the current lexical scope or compound statement.
  **L3785 CN**: 结束当前词法作用域或复合语句块。
- **L3786 EN**: Blank line separating nearby declarations or logic blocks.
  **L3786 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static DynamicType GetReturnType(const SpecificIntrinsicInterface &interface,`.
  **L3787 CN**: 继续一个多行参数列表、初始化器或聚合项：`static DynamicType GetReturnType(const SpecificIntrinsicInterface &interface,`。
- **L3788 EN**: Continues the surrounding expression or declaration: `const common::IntrinsicTypeDefaultKinds &defaults) {`.
  **L3788 CN**: 继续构造周围的表达式或声明：`const common::IntrinsicTypeDefaultKinds &defaults) {`。
- **L3789 EN**: Executes a standalone statement or declaration: `TypeCategory category{TypeCategory::Integer};`.
  **L3789 CN**: 执行一条独立语句或声明：`TypeCategory category{TypeCategory::Integer};`。
- **L3790 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L3790 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L3791 EN**: Introduces a switch dispatch label: `case KindCode::defaultIntegerKind:`.
  **L3791 CN**: 引入一个 switch 分发标签：`case KindCode::defaultIntegerKind:`。
- **L3792 EN**: Exits the nearest loop or switch statement.
  **L3792 CN**: 退出最近的循环或 switch 语句。

### Lines 3793-3816

````cpp
  case KindCode::doublePrecision:
  case KindCode::quadPrecision:
  case KindCode::defaultRealKind:
    category = TypeCategory::Real;
    break;
  default:
    CRASH_NO_CASE;
  }
  int kind{interface.result.kindCode == KindCode::doublePrecision
          ? defaults.doublePrecisionKind()
          : interface.result.kindCode == KindCode::quadPrecision
          ? defaults.quadPrecisionKind()
          : defaults.GetDefaultKind(category)};
  return DynamicType{category, kind};
}

// Probe the configured intrinsic procedure pattern tables in search of a
// match for a given procedure reference.
std::optional<SpecificCall> IntrinsicProcTable::Implementation::Probe(
    const CallCharacteristics &call, ActualArguments &arguments,
    FoldingContext &context) const {

  // All special cases handled here before the table probes below must
  // also be recognized as special names in IsIntrinsicSubroutine().
````
- **L3793 EN**: Introduces a switch dispatch label: `case KindCode::doublePrecision:`.
  **L3793 CN**: 引入一个 switch 分发标签：`case KindCode::doublePrecision:`。
- **L3794 EN**: Introduces a switch dispatch label: `case KindCode::quadPrecision:`.
  **L3794 CN**: 引入一个 switch 分发标签：`case KindCode::quadPrecision:`。
- **L3795 EN**: Introduces a switch dispatch label: `case KindCode::defaultRealKind:`.
  **L3795 CN**: 引入一个 switch 分发标签：`case KindCode::defaultRealKind:`。
- **L3796 EN**: Executes a standalone statement or declaration: `category = TypeCategory::Real;`.
  **L3796 CN**: 执行一条独立语句或声明：`category = TypeCategory::Real;`。
- **L3797 EN**: Exits the nearest loop or switch statement.
  **L3797 CN**: 退出最近的循环或 switch 语句。
- **L3798 EN**: Introduces a switch dispatch label: `default:`.
  **L3798 CN**: 引入一个 switch 分发标签：`default:`。
- **L3799 EN**: Executes a standalone statement or declaration: `CRASH_NO_CASE;`.
  **L3799 CN**: 执行一条独立语句或声明：`CRASH_NO_CASE;`。
- **L3800 EN**: Closes the current lexical scope or compound statement.
  **L3800 CN**: 结束当前词法作用域或复合语句块。
- **L3801 EN**: Continues the surrounding expression or declaration: `int kind{interface.result.kindCode == KindCode::doublePrecision`.
  **L3801 CN**: 继续构造周围的表达式或声明：`int kind{interface.result.kindCode == KindCode::doublePrecision`。
- **L3802 EN**: Continues logic associated with callable symbol `doublePrecisionKind`.
  **L3802 CN**: 继续与可调用符号 `doublePrecisionKind` 相关的逻辑。
- **L3803 EN**: Continues the surrounding expression or declaration: `: interface.result.kindCode == KindCode::quadPrecision`.
  **L3803 CN**: 继续构造周围的表达式或声明：`: interface.result.kindCode == KindCode::quadPrecision`。
- **L3804 EN**: Continues logic associated with callable symbol `quadPrecisionKind`.
  **L3804 CN**: 继续与可调用符号 `quadPrecisionKind` 相关的逻辑。
- **L3805 EN**: Executes a call or declaration centered on `defaults.GetDefaultKind`.
  **L3805 CN**: 执行以 `defaults.GetDefaultKind` 为核心的调用或声明。
- **L3806 EN**: Returns from the current function with `DynamicType{category, kind}`.
  **L3806 CN**: 以 `DynamicType{category, kind}` 从当前函数返回。
- **L3807 EN**: Closes the current lexical scope or compound statement.
  **L3807 CN**: 结束当前词法作用域或复合语句块。
- **L3808 EN**: Blank line separating nearby declarations or logic blocks.
  **L3808 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3809 EN**: Comment explains nearby logic, intent, or metadata: `Probe the configured intrinsic procedure pattern tables in search of a`.
  **L3809 CN**: 注释说明附近代码的逻辑、意图或元数据：`Probe the configured intrinsic procedure pattern tables in search of a`。
- **L3810 EN**: Comment explains nearby logic, intent, or metadata: `match for a given procedure reference.`.
  **L3810 CN**: 注释说明附近代码的逻辑、意图或元数据：`match for a given procedure reference.`。
- **L3811 EN**: Continues logic associated with callable symbol `Probe`.
  **L3811 CN**: 继续与可调用符号 `Probe` 相关的逻辑。
- **L3812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const CallCharacteristics &call, ActualArguments &arguments,`.
  **L3812 CN**: 继续一个多行参数列表、初始化器或聚合项：`const CallCharacteristics &call, ActualArguments &arguments,`。
- **L3813 EN**: Continues the surrounding expression or declaration: `FoldingContext &context) const {`.
  **L3813 CN**: 继续构造周围的表达式或声明：`FoldingContext &context) const {`。
- **L3814 EN**: Blank line separating nearby declarations or logic blocks.
  **L3814 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3815 EN**: Comment explains nearby logic, intent, or metadata: `All special cases handled here before the table probes below must`.
  **L3815 CN**: 注释说明附近代码的逻辑、意图或元数据：`All special cases handled here before the table probes below must`。
- **L3816 EN**: Comment explains nearby logic, intent, or metadata: `also be recognized as special names in IsIntrinsicSubroutine().`.
  **L3816 CN**: 注释说明附近代码的逻辑、意图或元数据：`also be recognized as special names in IsIntrinsicSubroutine().`。

### Lines 3817-3840

````cpp
  if (call.isSubroutineCall) {
    if (call.name == "__builtin_c_f_pointer") {
      return HandleC_F_Pointer(arguments, context);
    } else if (call.name == "__builtin_c_f_strpointer") {
      return HandleC_F_Strpointer(arguments, context);
    } else if (call.name == "random_seed") {
      int optionalCount{0};
      for (const auto &arg : arguments) {
        if (const auto *expr{arg->UnwrapExpr()}) {
          optionalCount +=
              Fortran::evaluate::MayBePassedAsAbsentOptional(*expr);
        }
      }
      if (arguments.size() - optionalCount > 1) {
        context.messages().Say(
            "RANDOM_SEED must have either 1 or no arguments"_err_en_US);
      }
    }
  } else { // function
    if (call.name == "__builtin_c_loc") {
      return HandleC_Loc(arguments, context);
    } else if (call.name == "__builtin_c_devloc") {
      return HandleC_Devloc(arguments, context);
    } else if (call.name == "null") {
````
- **L3817 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3817 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3818 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3818 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3819 EN**: Returns from the current function with `HandleC_F_Pointer(arguments, context)`.
  **L3819 CN**: 以 `HandleC_F_Pointer(arguments, context)` 从当前函数返回。
- **L3820 EN**: Transitions from the previous branch into an `else if` condition.
  **L3820 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3821 EN**: Returns from the current function with `HandleC_F_Strpointer(arguments, context)`.
  **L3821 CN**: 以 `HandleC_F_Strpointer(arguments, context)` 从当前函数返回。
- **L3822 EN**: Transitions from the previous branch into an `else if` condition.
  **L3822 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3823 EN**: Executes a standalone statement or declaration: `int optionalCount{0};`.
  **L3823 CN**: 执行一条独立语句或声明：`int optionalCount{0};`。
- **L3824 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3824 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3825 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3825 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3826 EN**: Continues the surrounding expression or declaration: `optionalCount +=`.
  **L3826 CN**: 继续构造周围的表达式或声明：`optionalCount +=`。
- **L3827 EN**: Executes a call or declaration centered on `Fortran::evaluate::MayBePassedAsAbsentOptional`.
  **L3827 CN**: 执行以 `Fortran::evaluate::MayBePassedAsAbsentOptional` 为核心的调用或声明。
- **L3828 EN**: Closes the current lexical scope or compound statement.
  **L3828 CN**: 结束当前词法作用域或复合语句块。
- **L3829 EN**: Closes the current lexical scope or compound statement.
  **L3829 CN**: 结束当前词法作用域或复合语句块。
- **L3830 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3830 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3831 EN**: Continues logic associated with callable symbol `messages`.
  **L3831 CN**: 继续与可调用符号 `messages` 相关的逻辑。
- **L3832 EN**: Executes a standalone statement or declaration: `"RANDOM_SEED must have either 1 or no arguments"_err_en_US);`.
  **L3832 CN**: 执行一条独立语句或声明：`"RANDOM_SEED must have either 1 or no arguments"_err_en_US);`。
- **L3833 EN**: Closes the current lexical scope or compound statement.
  **L3833 CN**: 结束当前词法作用域或复合语句块。
- **L3834 EN**: Closes the current lexical scope or compound statement.
  **L3834 CN**: 结束当前词法作用域或复合语句块。
- **L3835 EN**: Transitions from the previous branch into the alternative path.
  **L3835 CN**: 从前一个分支过渡到备选路径。
- **L3836 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3836 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3837 EN**: Returns from the current function with `HandleC_Loc(arguments, context)`.
  **L3837 CN**: 以 `HandleC_Loc(arguments, context)` 从当前函数返回。
- **L3838 EN**: Transitions from the previous branch into an `else if` condition.
  **L3838 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3839 EN**: Returns from the current function with `HandleC_Devloc(arguments, context)`.
  **L3839 CN**: 以 `HandleC_Devloc(arguments, context)` 从当前函数返回。
- **L3840 EN**: Transitions from the previous branch into an `else if` condition.
  **L3840 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 3841-3864

````cpp
      return HandleNull(arguments, context);
    } else if (call.name == "allocated") {
      if (context.languageFeatures().IsEnabled(
              common::LanguageFeature::AllocatedForAssociated) &&
          arguments.size() == 1 && arguments[0].has_value()) {
        auto &arg{*arguments[0]};
        if (const Expr<SomeType> *expr{arg.UnwrapExpr()};
            expr && IsObjectPointer(*expr)) {
          context.Warn(common::LanguageFeature::AllocatedForAssociated,
              arg.sourceLocation(),
              "Argument of ALLOCATED() should be an allocatable, but is instead an object pointer"_warn_en_US);
          // Treat ALLOCATED(ptr) as ASSOCIATED(ptr)
          CallCharacteristics newCall{"associated"};
          return Probe(newCall, arguments, context);
        }
      }
    }
  }

  // Find the specific subroutine and match the actual arguments against its
  // dummy argument patterns. If there are multiple specific subroutines with
  // the same name, try them in order.  If one matches, clear out the errors.
  // If none match, keep the messages from the form whose dummy argument
  // types and keywords best match the actual arguments supplied.
````
- **L3841 EN**: Returns from the current function with `HandleNull(arguments, context)`.
  **L3841 CN**: 以 `HandleNull(arguments, context)` 从当前函数返回。
- **L3842 EN**: Transitions from the previous branch into an `else if` condition.
  **L3842 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3843 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3843 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3844 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::AllocatedForAssociated) &&`.
  **L3844 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::AllocatedForAssociated) &&`。
- **L3845 EN**: Starts a function, method, lambda, or structured scope: `arguments.size() == 1 && arguments[0].has_value()) {`.
  **L3845 CN**: 开始一个函数、方法、lambda 或结构化作用域：`arguments.size() == 1 && arguments[0].has_value()) {`。
- **L3846 EN**: Executes a standalone statement or declaration: `auto &arg{*arguments[0]};`.
  **L3846 CN**: 执行一条独立语句或声明：`auto &arg{*arguments[0]};`。
- **L3847 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3847 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3848 EN**: Starts a function, method, lambda, or structured scope: `expr && IsObjectPointer(*expr)) {`.
  **L3848 CN**: 开始一个函数、方法、lambda 或结构化作用域：`expr && IsObjectPointer(*expr)) {`。
- **L3849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(common::LanguageFeature::AllocatedForAssociated,`.
  **L3849 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(common::LanguageFeature::AllocatedForAssociated,`。
- **L3850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arg.sourceLocation(),`.
  **L3850 CN**: 继续一个多行参数列表、初始化器或聚合项：`arg.sourceLocation(),`。
- **L3851 EN**: Executes a call or declaration centered on `ALLOCATED`.
  **L3851 CN**: 执行以 `ALLOCATED` 为核心的调用或声明。
- **L3852 EN**: Comment explains nearby logic, intent, or metadata: `Treat ALLOCATED(ptr) as ASSOCIATED(ptr)`.
  **L3852 CN**: 注释说明附近代码的逻辑、意图或元数据：`Treat ALLOCATED(ptr) as ASSOCIATED(ptr)`。
- **L3853 EN**: Executes a standalone statement or declaration: `CallCharacteristics newCall{"associated"};`.
  **L3853 CN**: 执行一条独立语句或声明：`CallCharacteristics newCall{"associated"};`。
- **L3854 EN**: Returns from the current function with `Probe(newCall, arguments, context)`.
  **L3854 CN**: 以 `Probe(newCall, arguments, context)` 从当前函数返回。
- **L3855 EN**: Closes the current lexical scope or compound statement.
  **L3855 CN**: 结束当前词法作用域或复合语句块。
- **L3856 EN**: Closes the current lexical scope or compound statement.
  **L3856 CN**: 结束当前词法作用域或复合语句块。
- **L3857 EN**: Closes the current lexical scope or compound statement.
  **L3857 CN**: 结束当前词法作用域或复合语句块。
- **L3858 EN**: Closes the current lexical scope or compound statement.
  **L3858 CN**: 结束当前词法作用域或复合语句块。
- **L3859 EN**: Blank line separating nearby declarations or logic blocks.
  **L3859 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3860 EN**: Comment explains nearby logic, intent, or metadata: `Find the specific subroutine and match the actual arguments against its`.
  **L3860 CN**: 注释说明附近代码的逻辑、意图或元数据：`Find the specific subroutine and match the actual arguments against its`。
- **L3861 EN**: Comment explains nearby logic, intent, or metadata: `dummy argument patterns. If there are multiple specific subroutines with`.
  **L3861 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummy argument patterns. If there are multiple specific subroutines with`。
- **L3862 EN**: Comment explains nearby logic, intent, or metadata: `the same name, try them in order.  If one matches, clear out the errors.`.
  **L3862 CN**: 注释说明附近代码的逻辑、意图或元数据：`the same name, try them in order.  If one matches, clear out the errors.`。
- **L3863 EN**: Comment explains nearby logic, intent, or metadata: `If none match, keep the messages from the form whose dummy argument`.
  **L3863 CN**: 注释说明附近代码的逻辑、意图或元数据：`If none match, keep the messages from the form whose dummy argument`。
- **L3864 EN**: Comment explains nearby logic, intent, or metadata: `types and keywords best match the actual arguments supplied.`.
  **L3864 CN**: 注释说明附近代码的逻辑、意图或元数据：`types and keywords best match the actual arguments supplied.`。

### Lines 3865-3888

````cpp
  if (call.isSubroutineCall) {
    const std::string &name{ResolveAlias(call.name)};
    auto subrRange{subroutines_.equal_range(name)};
    parser::Messages subrErrors;
    int bestScore{INT_MIN};
    parser::Messages localBuffer;
    parser::Messages *finalBuffer{context.messages().messages()};
    parser::ContextualMessages localMessages{
        context.messages().at(), finalBuffer ? &localBuffer : nullptr};
    FoldingContext localContext{context, localMessages};
    for (auto iter{subrRange.first}; iter != subrRange.second; ++iter) {
      if (auto specificCall{iter->second->Match(
              call, defaults_, arguments, localContext, builtinsScope_)}) {
        if (finalBuffer) {
          finalBuffer->Annex(std::move(localBuffer));
        }
        ApplySpecificChecks(*specificCall, context);
        return specificCall;
      }
      // Match failed.  Compute a score reflecting how well the actual
      // arguments correspond to this form's dummy arguments: count the
      // number of positional arguments whose type category matches the
      // corresponding dummy's expected categories, plus the number of
      // keyword arguments whose keyword name matches a dummy in this form,
````
- **L3865 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3865 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3866 EN**: Executes a call or declaration centered on `&name{ResolveAlias`.
  **L3866 CN**: 执行以 `&name{ResolveAlias` 为核心的调用或声明。
- **L3867 EN**: Executes a call or declaration centered on `subrRange{subroutines_.equal_range`.
  **L3867 CN**: 执行以 `subrRange{subroutines_.equal_range` 为核心的调用或声明。
- **L3868 EN**: Executes a standalone statement or declaration: `parser::Messages subrErrors;`.
  **L3868 CN**: 执行一条独立语句或声明：`parser::Messages subrErrors;`。
- **L3869 EN**: Executes a standalone statement or declaration: `int bestScore{INT_MIN};`.
  **L3869 CN**: 执行一条独立语句或声明：`int bestScore{INT_MIN};`。
- **L3870 EN**: Executes a standalone statement or declaration: `parser::Messages localBuffer;`.
  **L3870 CN**: 执行一条独立语句或声明：`parser::Messages localBuffer;`。
- **L3871 EN**: Executes a call or declaration centered on `*finalBuffer{context.messages`.
  **L3871 CN**: 执行以 `*finalBuffer{context.messages` 为核心的调用或声明。
- **L3872 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages localMessages{`.
  **L3872 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages localMessages{`。
- **L3873 EN**: Executes a call or declaration centered on `context.messages`.
  **L3873 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L3874 EN**: Executes a standalone statement or declaration: `FoldingContext localContext{context, localMessages};`.
  **L3874 CN**: 执行一条独立语句或声明：`FoldingContext localContext{context, localMessages};`。
- **L3875 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3875 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3876 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3876 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3877 EN**: Continues the surrounding expression or declaration: `call, defaults_, arguments, localContext, builtinsScope_)}) {`.
  **L3877 CN**: 继续构造周围的表达式或声明：`call, defaults_, arguments, localContext, builtinsScope_)}) {`。
- **L3878 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3878 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3879 EN**: Executes a call or declaration centered on `finalBuffer->Annex`.
  **L3879 CN**: 执行以 `finalBuffer->Annex` 为核心的调用或声明。
- **L3880 EN**: Closes the current lexical scope or compound statement.
  **L3880 CN**: 结束当前词法作用域或复合语句块。
- **L3881 EN**: Executes a call or declaration centered on `ApplySpecificChecks`.
  **L3881 CN**: 执行以 `ApplySpecificChecks` 为核心的调用或声明。
- **L3882 EN**: Returns from the current function with `specificCall`.
  **L3882 CN**: 以 `specificCall` 从当前函数返回。
- **L3883 EN**: Closes the current lexical scope or compound statement.
  **L3883 CN**: 结束当前词法作用域或复合语句块。
- **L3884 EN**: Comment explains nearby logic, intent, or metadata: `Match failed.  Compute a score reflecting how well the actual`.
  **L3884 CN**: 注释说明附近代码的逻辑、意图或元数据：`Match failed.  Compute a score reflecting how well the actual`。
- **L3885 EN**: Comment explains nearby logic, intent, or metadata: `arguments correspond to this form's dummy arguments: count the`.
  **L3885 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments correspond to this form's dummy arguments: count the`。
- **L3886 EN**: Comment explains nearby logic, intent, or metadata: `number of positional arguments whose type category matches the`.
  **L3886 CN**: 注释说明附近代码的逻辑、意图或元数据：`number of positional arguments whose type category matches the`。
- **L3887 EN**: Comment explains nearby logic, intent, or metadata: `corresponding dummy's expected categories, plus the number of`.
  **L3887 CN**: 注释说明附近代码的逻辑、意图或元数据：`corresponding dummy's expected categories, plus the number of`。
- **L3888 EN**: Comment explains nearby logic, intent, or metadata: `keyword arguments whose keyword name matches a dummy in this form,`.
  **L3888 CN**: 注释说明附近代码的逻辑、意图或元数据：`keyword arguments whose keyword name matches a dummy in this form,`。

### Lines 3889-3912

````cpp
      // minus the number of required dummies that cannot be satisfied by
      // the number of arguments provided.  Keep the error messages from
      // the form with the highest score, preferring an earlier form on
      // ties.
      const auto *iface{iter->second};
      int dummyCount{iface->CountArguments()};
      int numRequired{0};
      for (int j{0}; j < dummyCount; ++j) {
        if (iface->dummy[j].optionality == Optionality::required) {
          ++numRequired;
        }
      }
      int score{0};
      int positionalIndex{0};
      for (const auto &arg : arguments) {
        if (arg) {
          if (auto kw{arg->keyword()}) {
            for (int k{0}; k < dummyCount; ++k) {
              if (kw == iface->dummy[k].keyword) {
                ++score;
                break;
              }
            }
          } else {
````
- **L3889 EN**: Comment explains nearby logic, intent, or metadata: `minus the number of required dummies that cannot be satisfied by`.
  **L3889 CN**: 注释说明附近代码的逻辑、意图或元数据：`minus the number of required dummies that cannot be satisfied by`。
- **L3890 EN**: Comment explains nearby logic, intent, or metadata: `the number of arguments provided.  Keep the error messages from`.
  **L3890 CN**: 注释说明附近代码的逻辑、意图或元数据：`the number of arguments provided.  Keep the error messages from`。
- **L3891 EN**: Comment explains nearby logic, intent, or metadata: `the form with the highest score, preferring an earlier form on`.
  **L3891 CN**: 注释说明附近代码的逻辑、意图或元数据：`the form with the highest score, preferring an earlier form on`。
- **L3892 EN**: Comment explains nearby logic, intent, or metadata: `ties.`.
  **L3892 CN**: 注释说明附近代码的逻辑、意图或元数据：`ties.`。
- **L3893 EN**: Executes a standalone statement or declaration: `const auto *iface{iter->second};`.
  **L3893 CN**: 执行一条独立语句或声明：`const auto *iface{iter->second};`。
- **L3894 EN**: Executes a call or declaration centered on `dummyCount{iface->CountArguments`.
  **L3894 CN**: 执行以 `dummyCount{iface->CountArguments` 为核心的调用或声明。
- **L3895 EN**: Executes a standalone statement or declaration: `int numRequired{0};`.
  **L3895 CN**: 执行一条独立语句或声明：`int numRequired{0};`。
- **L3896 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3896 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3897 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3897 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3898 EN**: Executes a standalone statement or declaration: `++numRequired;`.
  **L3898 CN**: 执行一条独立语句或声明：`++numRequired;`。
- **L3899 EN**: Closes the current lexical scope or compound statement.
  **L3899 CN**: 结束当前词法作用域或复合语句块。
- **L3900 EN**: Closes the current lexical scope or compound statement.
  **L3900 CN**: 结束当前词法作用域或复合语句块。
- **L3901 EN**: Executes a standalone statement or declaration: `int score{0};`.
  **L3901 CN**: 执行一条独立语句或声明：`int score{0};`。
- **L3902 EN**: Executes a standalone statement or declaration: `int positionalIndex{0};`.
  **L3902 CN**: 执行一条独立语句或声明：`int positionalIndex{0};`。
- **L3903 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3903 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3904 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3904 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3905 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3905 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3906 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3906 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3907 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3907 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3908 EN**: Executes a standalone statement or declaration: `++score;`.
  **L3908 CN**: 执行一条独立语句或声明：`++score;`。
- **L3909 EN**: Exits the nearest loop or switch statement.
  **L3909 CN**: 退出最近的循环或 switch 语句。
- **L3910 EN**: Closes the current lexical scope or compound statement.
  **L3910 CN**: 结束当前词法作用域或复合语句块。
- **L3911 EN**: Closes the current lexical scope or compound statement.
  **L3911 CN**: 结束当前词法作用域或复合语句块。
- **L3912 EN**: Transitions from the previous branch into the alternative path.
  **L3912 CN**: 从前一个分支过渡到备选路径。

### Lines 3913-3936

````cpp
            if (positionalIndex < dummyCount) {
              if (auto type{arg->GetType()}) {
                if (iface->dummy[positionalIndex].typePattern.categorySet.test(
                        type->category())) {
                  ++score;
                }
              }
            }
            ++positionalIndex;
          }
        }
      }
      score -= std::max(0, numRequired - static_cast<int>(arguments.size()));
      if (score > bestScore) {
        bestScore = score;
        subrErrors = std::move(localBuffer);
      } else {
        localBuffer.clear();
      }
    }
    if (finalBuffer) {
      finalBuffer->Annex(std::move(subrErrors));
    }
    if (IsIntrinsicFunction(call.name) && !IsDualIntrinsic(call.name)) {
````
- **L3913 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3913 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3914 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3914 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3915 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3915 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3916 EN**: Starts a function, method, lambda, or structured scope: `type->category())) {`.
  **L3916 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type->category())) {`。
- **L3917 EN**: Executes a standalone statement or declaration: `++score;`.
  **L3917 CN**: 执行一条独立语句或声明：`++score;`。
- **L3918 EN**: Closes the current lexical scope or compound statement.
  **L3918 CN**: 结束当前词法作用域或复合语句块。
- **L3919 EN**: Closes the current lexical scope or compound statement.
  **L3919 CN**: 结束当前词法作用域或复合语句块。
- **L3920 EN**: Closes the current lexical scope or compound statement.
  **L3920 CN**: 结束当前词法作用域或复合语句块。
- **L3921 EN**: Executes a standalone statement or declaration: `++positionalIndex;`.
  **L3921 CN**: 执行一条独立语句或声明：`++positionalIndex;`。
- **L3922 EN**: Closes the current lexical scope or compound statement.
  **L3922 CN**: 结束当前词法作用域或复合语句块。
- **L3923 EN**: Closes the current lexical scope or compound statement.
  **L3923 CN**: 结束当前词法作用域或复合语句块。
- **L3924 EN**: Closes the current lexical scope or compound statement.
  **L3924 CN**: 结束当前词法作用域或复合语句块。
- **L3925 EN**: Executes a call or declaration centered on `std::max`.
  **L3925 CN**: 执行以 `std::max` 为核心的调用或声明。
- **L3926 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3926 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3927 EN**: Executes a standalone statement or declaration: `bestScore = score;`.
  **L3927 CN**: 执行一条独立语句或声明：`bestScore = score;`。
- **L3928 EN**: Executes a call or declaration centered on `std::move`.
  **L3928 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3929 EN**: Transitions from the previous branch into the alternative path.
  **L3929 CN**: 从前一个分支过渡到备选路径。
- **L3930 EN**: Executes a call or declaration centered on `localBuffer.clear`.
  **L3930 CN**: 执行以 `localBuffer.clear` 为核心的调用或声明。
- **L3931 EN**: Closes the current lexical scope or compound statement.
  **L3931 CN**: 结束当前词法作用域或复合语句块。
- **L3932 EN**: Closes the current lexical scope or compound statement.
  **L3932 CN**: 结束当前词法作用域或复合语句块。
- **L3933 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3933 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3934 EN**: Executes a call or declaration centered on `finalBuffer->Annex`.
  **L3934 CN**: 执行以 `finalBuffer->Annex` 为核心的调用或声明。
- **L3935 EN**: Closes the current lexical scope or compound statement.
  **L3935 CN**: 结束当前词法作用域或复合语句块。
- **L3936 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3936 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3937-3960

````cpp
      context.messages().Say(
          "Cannot use intrinsic function '%s' as a subroutine"_err_en_US,
          call.name);
    }
    return std::nullopt;
  }

  // Helper to avoid emitting errors before it is sure there is no match
  parser::Messages localBuffer;
  parser::Messages *finalBuffer{context.messages().messages()};
  parser::ContextualMessages localMessages{
      context.messages().at(), finalBuffer ? &localBuffer : nullptr};
  FoldingContext localContext{context, localMessages};
  auto matchOrBufferMessages{
      [&](const IntrinsicInterface &intrinsic,
          parser::Messages &buffer) -> std::optional<SpecificCall> {
        if (auto specificCall{intrinsic.Match(
                call, defaults_, arguments, localContext, builtinsScope_)}) {
          if (finalBuffer) {
            finalBuffer->Annex(std::move(localBuffer));
          }
          return specificCall;
        } else if (buffer.empty()) {
          buffer.Annex(std::move(localBuffer));
````
- **L3937 EN**: Continues logic associated with callable symbol `messages`.
  **L3937 CN**: 继续与可调用符号 `messages` 相关的逻辑。
- **L3938 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Cannot use intrinsic function '%s' as a subroutine"_err_en_US,`.
  **L3938 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Cannot use intrinsic function '%s' as a subroutine"_err_en_US,`。
- **L3939 EN**: Executes a standalone statement or declaration: `call.name);`.
  **L3939 CN**: 执行一条独立语句或声明：`call.name);`。
- **L3940 EN**: Closes the current lexical scope or compound statement.
  **L3940 CN**: 结束当前词法作用域或复合语句块。
- **L3941 EN**: Returns from the current function with `std::nullopt`.
  **L3941 CN**: 以 `std::nullopt` 从当前函数返回。
- **L3942 EN**: Closes the current lexical scope or compound statement.
  **L3942 CN**: 结束当前词法作用域或复合语句块。
- **L3943 EN**: Blank line separating nearby declarations or logic blocks.
  **L3943 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3944 EN**: Comment explains nearby logic, intent, or metadata: `Helper to avoid emitting errors before it is sure there is no match`.
  **L3944 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper to avoid emitting errors before it is sure there is no match`。
- **L3945 EN**: Executes a standalone statement or declaration: `parser::Messages localBuffer;`.
  **L3945 CN**: 执行一条独立语句或声明：`parser::Messages localBuffer;`。
- **L3946 EN**: Executes a call or declaration centered on `*finalBuffer{context.messages`.
  **L3946 CN**: 执行以 `*finalBuffer{context.messages` 为核心的调用或声明。
- **L3947 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages localMessages{`.
  **L3947 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages localMessages{`。
- **L3948 EN**: Executes a call or declaration centered on `context.messages`.
  **L3948 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L3949 EN**: Executes a standalone statement or declaration: `FoldingContext localContext{context, localMessages};`.
  **L3949 CN**: 执行一条独立语句或声明：`FoldingContext localContext{context, localMessages};`。
- **L3950 EN**: Continues the surrounding expression or declaration: `auto matchOrBufferMessages{`.
  **L3950 CN**: 继续构造周围的表达式或声明：`auto matchOrBufferMessages{`。
- **L3951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const IntrinsicInterface &intrinsic,`.
  **L3951 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const IntrinsicInterface &intrinsic,`。
- **L3952 EN**: Continues the surrounding expression or declaration: `parser::Messages &buffer) -> std::optional<SpecificCall> {`.
  **L3952 CN**: 继续构造周围的表达式或声明：`parser::Messages &buffer) -> std::optional<SpecificCall> {`。
- **L3953 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3953 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3954 EN**: Continues the surrounding expression or declaration: `call, defaults_, arguments, localContext, builtinsScope_)}) {`.
  **L3954 CN**: 继续构造周围的表达式或声明：`call, defaults_, arguments, localContext, builtinsScope_)}) {`。
- **L3955 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3955 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3956 EN**: Executes a call or declaration centered on `finalBuffer->Annex`.
  **L3956 CN**: 执行以 `finalBuffer->Annex` 为核心的调用或声明。
- **L3957 EN**: Closes the current lexical scope or compound statement.
  **L3957 CN**: 结束当前词法作用域或复合语句块。
- **L3958 EN**: Returns from the current function with `specificCall`.
  **L3958 CN**: 以 `specificCall` 从当前函数返回。
- **L3959 EN**: Transitions from the previous branch into an `else if` condition.
  **L3959 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3960 EN**: Executes a call or declaration centered on `buffer.Annex`.
  **L3960 CN**: 执行以 `buffer.Annex` 为核心的调用或声明。

### Lines 3961-3984

````cpp
        } else {
          // When there are multiple entries in the table for an
          // intrinsic that has multiple forms depending on the
          // presence of DIM=, use messages from a later entry if
          // the messages from an earlier entry complain about the
          // DIM= argument and it wasn't specified with a keyword.
          // Also prefer later messages when earlier ones are about
          // argument count mismatches, as a later entry that accepts
          // the right number of arguments will give more specific errors.
          bool preferLaterError{false};
          for (const auto &m : buffer.messages()) {
            std::string text{m.ToString()};
            if (text.find("'dim='") != std::string::npos) {
              bool hadDimKeyword{false};
              for (const auto &a : arguments) {
                if (a) {
                  if (auto kw{a->keyword()}; kw && kw == "dim") {
                    hadDimKeyword = true;
                    break;
                  }
                }
              }
              if (!hadDimKeyword) {
                preferLaterError = true;
````
- **L3961 EN**: Transitions from the previous branch into the alternative path.
  **L3961 CN**: 从前一个分支过渡到备选路径。
- **L3962 EN**: Comment explains nearby logic, intent, or metadata: `When there are multiple entries in the table for an`.
  **L3962 CN**: 注释说明附近代码的逻辑、意图或元数据：`When there are multiple entries in the table for an`。
- **L3963 EN**: Comment explains nearby logic, intent, or metadata: `intrinsic that has multiple forms depending on the`.
  **L3963 CN**: 注释说明附近代码的逻辑、意图或元数据：`intrinsic that has multiple forms depending on the`。
- **L3964 EN**: Comment explains nearby logic, intent, or metadata: `presence of DIM=, use messages from a later entry if`.
  **L3964 CN**: 注释说明附近代码的逻辑、意图或元数据：`presence of DIM=, use messages from a later entry if`。
- **L3965 EN**: Comment explains nearby logic, intent, or metadata: `the messages from an earlier entry complain about the`.
  **L3965 CN**: 注释说明附近代码的逻辑、意图或元数据：`the messages from an earlier entry complain about the`。
- **L3966 EN**: Comment explains nearby logic, intent, or metadata: `DIM= argument and it wasn't specified with a keyword.`.
  **L3966 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIM= argument and it wasn't specified with a keyword.`。
- **L3967 EN**: Comment explains nearby logic, intent, or metadata: `Also prefer later messages when earlier ones are about`.
  **L3967 CN**: 注释说明附近代码的逻辑、意图或元数据：`Also prefer later messages when earlier ones are about`。
- **L3968 EN**: Comment explains nearby logic, intent, or metadata: `argument count mismatches, as a later entry that accepts`.
  **L3968 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument count mismatches, as a later entry that accepts`。
- **L3969 EN**: Comment explains nearby logic, intent, or metadata: `the right number of arguments will give more specific errors.`.
  **L3969 CN**: 注释说明附近代码的逻辑、意图或元数据：`the right number of arguments will give more specific errors.`。
- **L3970 EN**: Executes a standalone statement or declaration: `bool preferLaterError{false};`.
  **L3970 CN**: 执行一条独立语句或声明：`bool preferLaterError{false};`。
- **L3971 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3971 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3972 EN**: Executes a call or declaration centered on `text{m.ToString`.
  **L3972 CN**: 执行以 `text{m.ToString` 为核心的调用或声明。
- **L3973 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3973 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3974 EN**: Executes a standalone statement or declaration: `bool hadDimKeyword{false};`.
  **L3974 CN**: 执行一条独立语句或声明：`bool hadDimKeyword{false};`。
- **L3975 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3975 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3976 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3976 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3977 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3977 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3978 EN**: Executes a standalone statement or declaration: `hadDimKeyword = true;`.
  **L3978 CN**: 执行一条独立语句或声明：`hadDimKeyword = true;`。
- **L3979 EN**: Exits the nearest loop or switch statement.
  **L3979 CN**: 退出最近的循环或 switch 语句。
- **L3980 EN**: Closes the current lexical scope or compound statement.
  **L3980 CN**: 结束当前词法作用域或复合语句块。
- **L3981 EN**: Closes the current lexical scope or compound statement.
  **L3981 CN**: 结束当前词法作用域或复合语句块。
- **L3982 EN**: Closes the current lexical scope or compound statement.
  **L3982 CN**: 结束当前词法作用域或复合语句块。
- **L3983 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3983 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3984 EN**: Executes a standalone statement or declaration: `preferLaterError = true;`.
  **L3984 CN**: 执行一条独立语句或声明：`preferLaterError = true;`。

### Lines 3985-4008

````cpp
              }
              break;
            } else if (text.find("too many actual arguments") !=
                std::string::npos) {
              // Prefer messages from an entry that matched the argument
              // count, as those will be more specific about what's wrong
              preferLaterError = true;
              break;
            }
          }
          if (preferLaterError) {
            buffer = std::move(localBuffer);
          }
          localBuffer.clear();
        }
        return std::nullopt;
      }};

  // Probe the generic intrinsic function table first; allow for
  // the use of a legacy alias.
  parser::Messages genericBuffer;
  const std::string &name{ResolveAlias(call.name)};
  auto genericRange{genericFuncs_.equal_range(name)};
  for (auto iter{genericRange.first}; iter != genericRange.second; ++iter) {
````
- **L3985 EN**: Closes the current lexical scope or compound statement.
  **L3985 CN**: 结束当前词法作用域或复合语句块。
- **L3986 EN**: Exits the nearest loop or switch statement.
  **L3986 CN**: 退出最近的循环或 switch 语句。
- **L3987 EN**: Transitions from the previous branch into an `else if` condition.
  **L3987 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3988 EN**: Continues the surrounding expression or declaration: `std::string::npos) {`.
  **L3988 CN**: 继续构造周围的表达式或声明：`std::string::npos) {`。
- **L3989 EN**: Comment explains nearby logic, intent, or metadata: `Prefer messages from an entry that matched the argument`.
  **L3989 CN**: 注释说明附近代码的逻辑、意图或元数据：`Prefer messages from an entry that matched the argument`。
- **L3990 EN**: Comment explains nearby logic, intent, or metadata: `count, as those will be more specific about what's wrong`.
  **L3990 CN**: 注释说明附近代码的逻辑、意图或元数据：`count, as those will be more specific about what's wrong`。
- **L3991 EN**: Executes a standalone statement or declaration: `preferLaterError = true;`.
  **L3991 CN**: 执行一条独立语句或声明：`preferLaterError = true;`。
- **L3992 EN**: Exits the nearest loop or switch statement.
  **L3992 CN**: 退出最近的循环或 switch 语句。
- **L3993 EN**: Closes the current lexical scope or compound statement.
  **L3993 CN**: 结束当前词法作用域或复合语句块。
- **L3994 EN**: Closes the current lexical scope or compound statement.
  **L3994 CN**: 结束当前词法作用域或复合语句块。
- **L3995 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3995 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3996 EN**: Executes a call or declaration centered on `std::move`.
  **L3996 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L3997 EN**: Closes the current lexical scope or compound statement.
  **L3997 CN**: 结束当前词法作用域或复合语句块。
- **L3998 EN**: Executes a call or declaration centered on `localBuffer.clear`.
  **L3998 CN**: 执行以 `localBuffer.clear` 为核心的调用或声明。
- **L3999 EN**: Closes the current lexical scope or compound statement.
  **L3999 CN**: 结束当前词法作用域或复合语句块。
- **L4000 EN**: Returns from the current function with `std::nullopt`.
  **L4000 CN**: 以 `std::nullopt` 从当前函数返回。
- **L4001 EN**: Executes a standalone statement or declaration: `}};`.
  **L4001 CN**: 执行一条独立语句或声明：`}};`。
- **L4002 EN**: Blank line separating nearby declarations or logic blocks.
  **L4002 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4003 EN**: Comment explains nearby logic, intent, or metadata: `Probe the generic intrinsic function table first; allow for`.
  **L4003 CN**: 注释说明附近代码的逻辑、意图或元数据：`Probe the generic intrinsic function table first; allow for`。
- **L4004 EN**: Comment explains nearby logic, intent, or metadata: `the use of a legacy alias.`.
  **L4004 CN**: 注释说明附近代码的逻辑、意图或元数据：`the use of a legacy alias.`。
- **L4005 EN**: Executes a standalone statement or declaration: `parser::Messages genericBuffer;`.
  **L4005 CN**: 执行一条独立语句或声明：`parser::Messages genericBuffer;`。
- **L4006 EN**: Executes a call or declaration centered on `&name{ResolveAlias`.
  **L4006 CN**: 执行以 `&name{ResolveAlias` 为核心的调用或声明。
- **L4007 EN**: Executes a call or declaration centered on `genericRange{genericFuncs_.equal_range`.
  **L4007 CN**: 执行以 `genericRange{genericFuncs_.equal_range` 为核心的调用或声明。
- **L4008 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4008 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 4009-4032

````cpp
    if (auto specificCall{
            matchOrBufferMessages(*iter->second, genericBuffer)}) {
      ApplySpecificChecks(*specificCall, context);
      return specificCall;
    }
  }

  // Probe the specific intrinsic function table next.
  parser::Messages specificBuffer;
  auto specificRange{specificFuncs_.equal_range(call.name)};
  for (auto specIter{specificRange.first}; specIter != specificRange.second;
       ++specIter) {
    // We only need to check the cases with distinct generic names.
    if (const char *genericName{specIter->second->generic}) {
      if (auto specificCall{
              matchOrBufferMessages(*specIter->second, specificBuffer)}) {
        if (!specIter->second->useGenericAndForceResultType) {
          specificCall->specificIntrinsic.name = genericName;
        }
        specificCall->specificIntrinsic.isRestrictedSpecific =
            specIter->second->isRestrictedSpecific;
        // TODO test feature AdditionalIntrinsics, warn on nonstandard
        // specifics with DoublePrecisionComplex arguments.
        return specificCall;
````
- **L4009 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4009 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4010 EN**: Starts a function, method, lambda, or structured scope: `matchOrBufferMessages(*iter->second, genericBuffer)}) {`.
  **L4010 CN**: 开始一个函数、方法、lambda 或结构化作用域：`matchOrBufferMessages(*iter->second, genericBuffer)}) {`。
- **L4011 EN**: Executes a call or declaration centered on `ApplySpecificChecks`.
  **L4011 CN**: 执行以 `ApplySpecificChecks` 为核心的调用或声明。
- **L4012 EN**: Returns from the current function with `specificCall`.
  **L4012 CN**: 以 `specificCall` 从当前函数返回。
- **L4013 EN**: Closes the current lexical scope or compound statement.
  **L4013 CN**: 结束当前词法作用域或复合语句块。
- **L4014 EN**: Closes the current lexical scope or compound statement.
  **L4014 CN**: 结束当前词法作用域或复合语句块。
- **L4015 EN**: Blank line separating nearby declarations or logic blocks.
  **L4015 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4016 EN**: Comment explains nearby logic, intent, or metadata: `Probe the specific intrinsic function table next.`.
  **L4016 CN**: 注释说明附近代码的逻辑、意图或元数据：`Probe the specific intrinsic function table next.`。
- **L4017 EN**: Executes a standalone statement or declaration: `parser::Messages specificBuffer;`.
  **L4017 CN**: 执行一条独立语句或声明：`parser::Messages specificBuffer;`。
- **L4018 EN**: Executes a call or declaration centered on `specificRange{specificFuncs_.equal_range`.
  **L4018 CN**: 执行以 `specificRange{specificFuncs_.equal_range` 为核心的调用或声明。
- **L4019 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4019 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4020 EN**: Continues the surrounding expression or declaration: `++specIter) {`.
  **L4020 CN**: 继续构造周围的表达式或声明：`++specIter) {`。
- **L4021 EN**: Comment explains nearby logic, intent, or metadata: `We only need to check the cases with distinct generic names.`.
  **L4021 CN**: 注释说明附近代码的逻辑、意图或元数据：`We only need to check the cases with distinct generic names.`。
- **L4022 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4022 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4023 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4023 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4024 EN**: Starts a function, method, lambda, or structured scope: `matchOrBufferMessages(*specIter->second, specificBuffer)}) {`.
  **L4024 CN**: 开始一个函数、方法、lambda 或结构化作用域：`matchOrBufferMessages(*specIter->second, specificBuffer)}) {`。
- **L4025 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4025 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4026 EN**: Executes a standalone statement or declaration: `specificCall->specificIntrinsic.name = genericName;`.
  **L4026 CN**: 执行一条独立语句或声明：`specificCall->specificIntrinsic.name = genericName;`。
- **L4027 EN**: Closes the current lexical scope or compound statement.
  **L4027 CN**: 结束当前词法作用域或复合语句块。
- **L4028 EN**: Continues the surrounding expression or declaration: `specificCall->specificIntrinsic.isRestrictedSpecific =`.
  **L4028 CN**: 继续构造周围的表达式或声明：`specificCall->specificIntrinsic.isRestrictedSpecific =`。
- **L4029 EN**: Executes a standalone statement or declaration: `specIter->second->isRestrictedSpecific;`.
  **L4029 CN**: 执行一条独立语句或声明：`specIter->second->isRestrictedSpecific;`。
- **L4030 EN**: Comment records a pending task or caution: `TODO test feature AdditionalIntrinsics, warn on nonstandard`.
  **L4030 CN**: 注释记录待办事项或注意点：`TODO test feature AdditionalIntrinsics, warn on nonstandard`。
- **L4031 EN**: Comment explains nearby logic, intent, or metadata: `specifics with DoublePrecisionComplex arguments.`.
  **L4031 CN**: 注释说明附近代码的逻辑、意图或元数据：`specifics with DoublePrecisionComplex arguments.`。
- **L4032 EN**: Returns from the current function with `specificCall`.
  **L4032 CN**: 以 `specificCall` 从当前函数返回。

### Lines 4033-4056

````cpp
      }
    }
  }

  // If there was no exact match with a specific, try to match the related
  // generic and convert the result to the specific required type.
  if (context.languageFeatures().IsEnabled(common::LanguageFeature::
              UseGenericIntrinsicWhenSpecificDoesntMatch)) {
    for (auto specIter{specificRange.first}; specIter != specificRange.second;
         ++specIter) {
      // We only need to check the cases with distinct generic names.
      if (const char *genericName{specIter->second->generic}) {
        if (specIter->second->useGenericAndForceResultType) {
          auto genericRange{genericFuncs_.equal_range(genericName)};
          for (auto genIter{genericRange.first}; genIter != genericRange.second;
               ++genIter) {
            if (auto specificCall{
                    matchOrBufferMessages(*genIter->second, specificBuffer)}) {
              // Force the call result type to the specific intrinsic result
              // type, if possible.
              DynamicType genericType{
                  DEREF(specificCall->specificIntrinsic.characteristics.value()
                            .functionResult.value()
                            .GetTypeAndShape())
````
- **L4033 EN**: Closes the current lexical scope or compound statement.
  **L4033 CN**: 结束当前词法作用域或复合语句块。
- **L4034 EN**: Closes the current lexical scope or compound statement.
  **L4034 CN**: 结束当前词法作用域或复合语句块。
- **L4035 EN**: Closes the current lexical scope or compound statement.
  **L4035 CN**: 结束当前词法作用域或复合语句块。
- **L4036 EN**: Blank line separating nearby declarations or logic blocks.
  **L4036 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4037 EN**: Comment explains nearby logic, intent, or metadata: `If there was no exact match with a specific, try to match the related`.
  **L4037 CN**: 注释说明附近代码的逻辑、意图或元数据：`If there was no exact match with a specific, try to match the related`。
- **L4038 EN**: Comment explains nearby logic, intent, or metadata: `generic and convert the result to the specific required type.`.
  **L4038 CN**: 注释说明附近代码的逻辑、意图或元数据：`generic and convert the result to the specific required type.`。
- **L4039 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4039 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4040 EN**: Continues the surrounding expression or declaration: `UseGenericIntrinsicWhenSpecificDoesntMatch)) {`.
  **L4040 CN**: 继续构造周围的表达式或声明：`UseGenericIntrinsicWhenSpecificDoesntMatch)) {`。
- **L4041 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4041 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4042 EN**: Continues the surrounding expression or declaration: `++specIter) {`.
  **L4042 CN**: 继续构造周围的表达式或声明：`++specIter) {`。
- **L4043 EN**: Comment explains nearby logic, intent, or metadata: `We only need to check the cases with distinct generic names.`.
  **L4043 CN**: 注释说明附近代码的逻辑、意图或元数据：`We only need to check the cases with distinct generic names.`。
- **L4044 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4044 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4045 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4045 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4046 EN**: Executes a call or declaration centered on `genericRange{genericFuncs_.equal_range`.
  **L4046 CN**: 执行以 `genericRange{genericFuncs_.equal_range` 为核心的调用或声明。
- **L4047 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4047 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4048 EN**: Continues the surrounding expression or declaration: `++genIter) {`.
  **L4048 CN**: 继续构造周围的表达式或声明：`++genIter) {`。
- **L4049 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4049 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4050 EN**: Starts a function, method, lambda, or structured scope: `matchOrBufferMessages(*genIter->second, specificBuffer)}) {`.
  **L4050 CN**: 开始一个函数、方法、lambda 或结构化作用域：`matchOrBufferMessages(*genIter->second, specificBuffer)}) {`。
- **L4051 EN**: Comment explains nearby logic, intent, or metadata: `Force the call result type to the specific intrinsic result`.
  **L4051 CN**: 注释说明附近代码的逻辑、意图或元数据：`Force the call result type to the specific intrinsic result`。
- **L4052 EN**: Comment explains nearby logic, intent, or metadata: `type, if possible.`.
  **L4052 CN**: 注释说明附近代码的逻辑、意图或元数据：`type, if possible.`。
- **L4053 EN**: Continues the surrounding expression or declaration: `DynamicType genericType{`.
  **L4053 CN**: 继续构造周围的表达式或声明：`DynamicType genericType{`。
- **L4054 EN**: Continues logic associated with callable symbol `DEREF`.
  **L4054 CN**: 继续与可调用符号 `DEREF` 相关的逻辑。
- **L4055 EN**: Continues logic associated with callable symbol `value`.
  **L4055 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L4056 EN**: Continues logic associated with callable symbol `GetTypeAndShape`.
  **L4056 CN**: 继续与可调用符号 `GetTypeAndShape` 相关的逻辑。

### Lines 4057-4080

````cpp
                      .type()};
              DynamicType newType{GetReturnType(*specIter->second, defaults_)};
              if (genericType.category() == newType.category() ||
                  ((genericType.category() == TypeCategory::Integer ||
                       genericType.category() == TypeCategory::Real) &&
                      (newType.category() == TypeCategory::Integer ||
                          newType.category() == TypeCategory::Real))) {
                context.Warn(common::LanguageFeature::
                                 UseGenericIntrinsicWhenSpecificDoesntMatch,
                    "Argument types do not match specific intrinsic '%s' requirements; using '%s' generic instead and converting the result to %s if needed"_port_en_US,
                    call.name, genericName, newType.AsFortran());
                specificCall->specificIntrinsic.name = call.name;
                specificCall->specificIntrinsic.characteristics.value()
                    .functionResult.value()
                    .SetType(newType);
                return specificCall;
              }
            }
          }
        }
      }
    }
  }

````
- **L4057 EN**: Executes a call or declaration centered on `.type`.
  **L4057 CN**: 执行以 `.type` 为核心的调用或声明。
- **L4058 EN**: Executes a call or declaration centered on `newType{GetReturnType`.
  **L4058 CN**: 执行以 `newType{GetReturnType` 为核心的调用或声明。
- **L4059 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4059 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4060 EN**: Continues logic associated with callable symbol `category`.
  **L4060 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L4061 EN**: Continues logic associated with callable symbol `category`.
  **L4061 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L4062 EN**: Continues logic associated with callable symbol `category`.
  **L4062 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L4063 EN**: Starts a function, method, lambda, or structured scope: `newType.category() == TypeCategory::Real))) {`.
  **L4063 CN**: 开始一个函数、方法、lambda 或结构化作用域：`newType.category() == TypeCategory::Real))) {`。
- **L4064 EN**: Continues logic associated with callable symbol `Warn`.
  **L4064 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L4065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UseGenericIntrinsicWhenSpecificDoesntMatch,`.
  **L4065 CN**: 继续一个多行参数列表、初始化器或聚合项：`UseGenericIntrinsicWhenSpecificDoesntMatch,`。
- **L4066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Argument types do not match specific intrinsic '%s' requirements; using '%s' generic instead and converting the result to %s if needed"_port_en_US,`.
  **L4066 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Argument types do not match specific intrinsic '%s' requirements; using '%s' generic instead and converting the result to %s if needed"_port_en_US,`。
- **L4067 EN**: Executes a call or declaration centered on `newType.AsFortran`.
  **L4067 CN**: 执行以 `newType.AsFortran` 为核心的调用或声明。
- **L4068 EN**: Executes a standalone statement or declaration: `specificCall->specificIntrinsic.name = call.name;`.
  **L4068 CN**: 执行一条独立语句或声明：`specificCall->specificIntrinsic.name = call.name;`。
- **L4069 EN**: Continues logic associated with callable symbol `value`.
  **L4069 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L4070 EN**: Continues logic associated with callable symbol `value`.
  **L4070 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L4071 EN**: Executes a call or declaration centered on `.SetType`.
  **L4071 CN**: 执行以 `.SetType` 为核心的调用或声明。
- **L4072 EN**: Returns from the current function with `specificCall`.
  **L4072 CN**: 以 `specificCall` 从当前函数返回。
- **L4073 EN**: Closes the current lexical scope or compound statement.
  **L4073 CN**: 结束当前词法作用域或复合语句块。
- **L4074 EN**: Closes the current lexical scope or compound statement.
  **L4074 CN**: 结束当前词法作用域或复合语句块。
- **L4075 EN**: Closes the current lexical scope or compound statement.
  **L4075 CN**: 结束当前词法作用域或复合语句块。
- **L4076 EN**: Closes the current lexical scope or compound statement.
  **L4076 CN**: 结束当前词法作用域或复合语句块。
- **L4077 EN**: Closes the current lexical scope or compound statement.
  **L4077 CN**: 结束当前词法作用域或复合语句块。
- **L4078 EN**: Closes the current lexical scope or compound statement.
  **L4078 CN**: 结束当前词法作用域或复合语句块。
- **L4079 EN**: Closes the current lexical scope or compound statement.
  **L4079 CN**: 结束当前词法作用域或复合语句块。
- **L4080 EN**: Blank line separating nearby declarations or logic blocks.
  **L4080 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 4081-4104

````cpp
  if (specificBuffer.empty() && genericBuffer.empty() &&
      IsIntrinsicSubroutine(call.name) && !IsDualIntrinsic(call.name)) {
    context.messages().Say(
        "Cannot use intrinsic subroutine '%s' as a function"_err_en_US,
        call.name);
  }

  // No match; report the right errors, if any
  if (finalBuffer) {
    if (specificBuffer.empty()) {
      finalBuffer->Annex(std::move(genericBuffer));
    } else {
      finalBuffer->Annex(std::move(specificBuffer));
    }
  }
  return std::nullopt;
}

std::optional<SpecificIntrinsicFunctionInterface>
IntrinsicProcTable::Implementation::IsSpecificIntrinsicFunction(
    const std::string &name) const {
  auto specificRange{specificFuncs_.equal_range(name)};
  for (auto iter{specificRange.first}; iter != specificRange.second; ++iter) {
    const SpecificIntrinsicInterface &specific{*iter->second};
````
- **L4081 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4081 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4082 EN**: Starts a function, method, lambda, or structured scope: `IsIntrinsicSubroutine(call.name) && !IsDualIntrinsic(call.name)) {`.
  **L4082 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsIntrinsicSubroutine(call.name) && !IsDualIntrinsic(call.name)) {`。
- **L4083 EN**: Continues logic associated with callable symbol `messages`.
  **L4083 CN**: 继续与可调用符号 `messages` 相关的逻辑。
- **L4084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Cannot use intrinsic subroutine '%s' as a function"_err_en_US,`.
  **L4084 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Cannot use intrinsic subroutine '%s' as a function"_err_en_US,`。
- **L4085 EN**: Executes a standalone statement or declaration: `call.name);`.
  **L4085 CN**: 执行一条独立语句或声明：`call.name);`。
- **L4086 EN**: Closes the current lexical scope or compound statement.
  **L4086 CN**: 结束当前词法作用域或复合语句块。
- **L4087 EN**: Blank line separating nearby declarations or logic blocks.
  **L4087 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4088 EN**: Comment explains nearby logic, intent, or metadata: `No match; report the right errors, if any`.
  **L4088 CN**: 注释说明附近代码的逻辑、意图或元数据：`No match; report the right errors, if any`。
- **L4089 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4089 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4090 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4090 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4091 EN**: Executes a call or declaration centered on `finalBuffer->Annex`.
  **L4091 CN**: 执行以 `finalBuffer->Annex` 为核心的调用或声明。
- **L4092 EN**: Transitions from the previous branch into the alternative path.
  **L4092 CN**: 从前一个分支过渡到备选路径。
- **L4093 EN**: Executes a call or declaration centered on `finalBuffer->Annex`.
  **L4093 CN**: 执行以 `finalBuffer->Annex` 为核心的调用或声明。
- **L4094 EN**: Closes the current lexical scope or compound statement.
  **L4094 CN**: 结束当前词法作用域或复合语句块。
- **L4095 EN**: Closes the current lexical scope or compound statement.
  **L4095 CN**: 结束当前词法作用域或复合语句块。
- **L4096 EN**: Returns from the current function with `std::nullopt`.
  **L4096 CN**: 以 `std::nullopt` 从当前函数返回。
- **L4097 EN**: Closes the current lexical scope or compound statement.
  **L4097 CN**: 结束当前词法作用域或复合语句块。
- **L4098 EN**: Blank line separating nearby declarations or logic blocks.
  **L4098 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4099 EN**: Continues the surrounding expression or declaration: `std::optional<SpecificIntrinsicFunctionInterface>`.
  **L4099 CN**: 继续构造周围的表达式或声明：`std::optional<SpecificIntrinsicFunctionInterface>`。
- **L4100 EN**: Continues logic associated with callable symbol `IsSpecificIntrinsicFunction`.
  **L4100 CN**: 继续与可调用符号 `IsSpecificIntrinsicFunction` 相关的逻辑。
- **L4101 EN**: Continues the surrounding expression or declaration: `const std::string &name) const {`.
  **L4101 CN**: 继续构造周围的表达式或声明：`const std::string &name) const {`。
- **L4102 EN**: Executes a call or declaration centered on `specificRange{specificFuncs_.equal_range`.
  **L4102 CN**: 执行以 `specificRange{specificFuncs_.equal_range` 为核心的调用或声明。
- **L4103 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4103 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4104 EN**: Executes a standalone statement or declaration: `const SpecificIntrinsicInterface &specific{*iter->second};`.
  **L4104 CN**: 执行一条独立语句或声明：`const SpecificIntrinsicInterface &specific{*iter->second};`。

### Lines 4105-4128

````cpp
    std::string genericName{name};
    if (specific.generic) {
      genericName = std::string(specific.generic);
    }
    characteristics::FunctionResult fResult{GetSpecificType(specific.result)};
    characteristics::DummyArguments args;
    int dummies{specific.CountArguments()};
    for (int j{0}; j < dummies; ++j) {
      characteristics::DummyDataObject dummy{
          GetSpecificType(specific.dummy[j].typePattern)};
      dummy.intent = specific.dummy[j].intent;
      args.emplace_back(
          std::string{specific.dummy[j].keyword}, std::move(dummy));
    }
    characteristics::Procedure::Attrs attrs;
    attrs.set(characteristics::Procedure::Attr::Pure)
        .set(characteristics::Procedure::Attr::Elemental);
    characteristics::Procedure chars{
        std::move(fResult), std::move(args), attrs};
    return SpecificIntrinsicFunctionInterface{
        std::move(chars), genericName, specific.isRestrictedSpecific};
  }
  return std::nullopt;
}
````
- **L4105 EN**: Executes a standalone statement or declaration: `std::string genericName{name};`.
  **L4105 CN**: 执行一条独立语句或声明：`std::string genericName{name};`。
- **L4106 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4106 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4107 EN**: Executes a call or declaration centered on `std::string`.
  **L4107 CN**: 执行以 `std::string` 为核心的调用或声明。
- **L4108 EN**: Closes the current lexical scope or compound statement.
  **L4108 CN**: 结束当前词法作用域或复合语句块。
- **L4109 EN**: Executes a call or declaration centered on `fResult{GetSpecificType`.
  **L4109 CN**: 执行以 `fResult{GetSpecificType` 为核心的调用或声明。
- **L4110 EN**: Executes a standalone statement or declaration: `characteristics::DummyArguments args;`.
  **L4110 CN**: 执行一条独立语句或声明：`characteristics::DummyArguments args;`。
- **L4111 EN**: Executes a call or declaration centered on `dummies{specific.CountArguments`.
  **L4111 CN**: 执行以 `dummies{specific.CountArguments` 为核心的调用或声明。
- **L4112 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4112 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4113 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject dummy{`.
  **L4113 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject dummy{`。
- **L4114 EN**: Executes a call or declaration centered on `GetSpecificType`.
  **L4114 CN**: 执行以 `GetSpecificType` 为核心的调用或声明。
- **L4115 EN**: Executes a standalone statement or declaration: `dummy.intent = specific.dummy[j].intent;`.
  **L4115 CN**: 执行一条独立语句或声明：`dummy.intent = specific.dummy[j].intent;`。
- **L4116 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L4116 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L4117 EN**: Executes a call or declaration centered on `std::move`.
  **L4117 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L4118 EN**: Closes the current lexical scope or compound statement.
  **L4118 CN**: 结束当前词法作用域或复合语句块。
- **L4119 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attrs attrs;`.
  **L4119 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attrs attrs;`。
- **L4120 EN**: Continues logic associated with callable symbol `set`.
  **L4120 CN**: 继续与可调用符号 `set` 相关的逻辑。
- **L4121 EN**: Executes a call or declaration centered on `.set`.
  **L4121 CN**: 执行以 `.set` 为核心的调用或声明。
- **L4122 EN**: Continues the surrounding expression or declaration: `characteristics::Procedure chars{`.
  **L4122 CN**: 继续构造周围的表达式或声明：`characteristics::Procedure chars{`。
- **L4123 EN**: Executes a call or declaration centered on `std::move`.
  **L4123 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L4124 EN**: Returns from the current function with `SpecificIntrinsicFunctionInterface{`.
  **L4124 CN**: 以 `SpecificIntrinsicFunctionInterface{` 从当前函数返回。
- **L4125 EN**: Executes a call or declaration centered on `std::move`.
  **L4125 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L4126 EN**: Closes the current lexical scope or compound statement.
  **L4126 CN**: 结束当前词法作用域或复合语句块。
- **L4127 EN**: Returns from the current function with `std::nullopt`.
  **L4127 CN**: 以 `std::nullopt` 从当前函数返回。
- **L4128 EN**: Closes the current lexical scope or compound statement.
  **L4128 CN**: 结束当前词法作用域或复合语句块。

### Lines 4129-4152

````cpp

DynamicType IntrinsicProcTable::Implementation::GetSpecificType(
    const TypePattern &pattern) const {
  const CategorySet &set{pattern.categorySet};
  CHECK(set.count() == 1);
  TypeCategory category{set.LeastElement().value()};
  if (pattern.kindCode == KindCode::doublePrecision) {
    return DynamicType{category, defaults_.doublePrecisionKind()};
  } else if (pattern.kindCode == KindCode::quadPrecision) {
    return DynamicType{category, defaults_.quadPrecisionKind()};
  } else if (category == TypeCategory::Character) {
    // All character arguments to specific intrinsic functions are
    // assumed-length.
    return DynamicType{defaults_.GetDefaultKind(category), assumedLen_};
  } else {
    return DynamicType{category, defaults_.GetDefaultKind(category)};
  }
}

IntrinsicProcTable::~IntrinsicProcTable() = default;

IntrinsicProcTable IntrinsicProcTable::Configure(
    const common::IntrinsicTypeDefaultKinds &defaults) {
  IntrinsicProcTable result;
````
- **L4129 EN**: Blank line separating nearby declarations or logic blocks.
  **L4129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4130 EN**: Continues logic associated with callable symbol `GetSpecificType`.
  **L4130 CN**: 继续与可调用符号 `GetSpecificType` 相关的逻辑。
- **L4131 EN**: Continues the surrounding expression or declaration: `const TypePattern &pattern) const {`.
  **L4131 CN**: 继续构造周围的表达式或声明：`const TypePattern &pattern) const {`。
- **L4132 EN**: Executes a standalone statement or declaration: `const CategorySet &set{pattern.categorySet};`.
  **L4132 CN**: 执行一条独立语句或声明：`const CategorySet &set{pattern.categorySet};`。
- **L4133 EN**: Executes a call or declaration centered on `CHECK`.
  **L4133 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L4134 EN**: Executes a call or declaration centered on `category{set.LeastElement`.
  **L4134 CN**: 执行以 `category{set.LeastElement` 为核心的调用或声明。
- **L4135 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4135 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4136 EN**: Returns from the current function with `DynamicType{category, defaults_.doublePrecisionKind()}`.
  **L4136 CN**: 以 `DynamicType{category, defaults_.doublePrecisionKind()}` 从当前函数返回。
- **L4137 EN**: Transitions from the previous branch into an `else if` condition.
  **L4137 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4138 EN**: Returns from the current function with `DynamicType{category, defaults_.quadPrecisionKind()}`.
  **L4138 CN**: 以 `DynamicType{category, defaults_.quadPrecisionKind()}` 从当前函数返回。
- **L4139 EN**: Transitions from the previous branch into an `else if` condition.
  **L4139 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4140 EN**: Comment explains nearby logic, intent, or metadata: `All character arguments to specific intrinsic functions are`.
  **L4140 CN**: 注释说明附近代码的逻辑、意图或元数据：`All character arguments to specific intrinsic functions are`。
- **L4141 EN**: Comment explains nearby logic, intent, or metadata: `assumed-length.`.
  **L4141 CN**: 注释说明附近代码的逻辑、意图或元数据：`assumed-length.`。
- **L4142 EN**: Returns from the current function with `DynamicType{defaults_.GetDefaultKind(category), assumedLen_}`.
  **L4142 CN**: 以 `DynamicType{defaults_.GetDefaultKind(category), assumedLen_}` 从当前函数返回。
- **L4143 EN**: Transitions from the previous branch into the alternative path.
  **L4143 CN**: 从前一个分支过渡到备选路径。
- **L4144 EN**: Returns from the current function with `DynamicType{category, defaults_.GetDefaultKind(category)}`.
  **L4144 CN**: 以 `DynamicType{category, defaults_.GetDefaultKind(category)}` 从当前函数返回。
- **L4145 EN**: Closes the current lexical scope or compound statement.
  **L4145 CN**: 结束当前词法作用域或复合语句块。
- **L4146 EN**: Closes the current lexical scope or compound statement.
  **L4146 CN**: 结束当前词法作用域或复合语句块。
- **L4147 EN**: Blank line separating nearby declarations or logic blocks.
  **L4147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4148 EN**: Executes a call or declaration centered on `IntrinsicProcTable::~IntrinsicProcTable`.
  **L4148 CN**: 执行以 `IntrinsicProcTable::~IntrinsicProcTable` 为核心的调用或声明。
- **L4149 EN**: Blank line separating nearby declarations or logic blocks.
  **L4149 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4150 EN**: Continues logic associated with callable symbol `Configure`.
  **L4150 CN**: 继续与可调用符号 `Configure` 相关的逻辑。
- **L4151 EN**: Continues the surrounding expression or declaration: `const common::IntrinsicTypeDefaultKinds &defaults) {`.
  **L4151 CN**: 继续构造周围的表达式或声明：`const common::IntrinsicTypeDefaultKinds &defaults) {`。
- **L4152 EN**: Executes a standalone statement or declaration: `IntrinsicProcTable result;`.
  **L4152 CN**: 执行一条独立语句或声明：`IntrinsicProcTable result;`。

### Lines 4153-4176

````cpp
  result.impl_ = std::make_unique<IntrinsicProcTable::Implementation>(defaults);
  return result;
}

void IntrinsicProcTable::SupplyBuiltins(
    const semantics::Scope &builtins) const {
  DEREF(impl_.get()).SupplyBuiltins(builtins);
}

bool IntrinsicProcTable::IsIntrinsic(const std::string &name) const {
  return DEREF(impl_.get()).IsIntrinsic(name);
}
bool IntrinsicProcTable::IsIntrinsicFunction(const std::string &name) const {
  return DEREF(impl_.get()).IsIntrinsicFunction(name);
}
bool IntrinsicProcTable::IsIntrinsicSubroutine(const std::string &name) const {
  return DEREF(impl_.get()).IsIntrinsicSubroutine(name);
}
bool IntrinsicProcTable::IsDualIntrinsic(const std::string &name) const {
  return DEREF(impl_.get()).IsDualIntrinsic(name);
}

IntrinsicClass IntrinsicProcTable::GetIntrinsicClass(
    const std::string &name) const {
````
- **L4153 EN**: Executes a call or declaration centered on `std::make_unique<IntrinsicProcTable::Implementation>`.
  **L4153 CN**: 执行以 `std::make_unique<IntrinsicProcTable::Implementation>` 为核心的调用或声明。
- **L4154 EN**: Returns from the current function with `result`.
  **L4154 CN**: 以 `result` 从当前函数返回。
- **L4155 EN**: Closes the current lexical scope or compound statement.
  **L4155 CN**: 结束当前词法作用域或复合语句块。
- **L4156 EN**: Blank line separating nearby declarations or logic blocks.
  **L4156 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4157 EN**: Continues logic associated with callable symbol `SupplyBuiltins`.
  **L4157 CN**: 继续与可调用符号 `SupplyBuiltins` 相关的逻辑。
- **L4158 EN**: Continues the surrounding expression or declaration: `const semantics::Scope &builtins) const {`.
  **L4158 CN**: 继续构造周围的表达式或声明：`const semantics::Scope &builtins) const {`。
- **L4159 EN**: Executes a call or declaration centered on `DEREF`.
  **L4159 CN**: 执行以 `DEREF` 为核心的调用或声明。
- **L4160 EN**: Closes the current lexical scope or compound statement.
  **L4160 CN**: 结束当前词法作用域或复合语句块。
- **L4161 EN**: Blank line separating nearby declarations or logic blocks.
  **L4161 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4162 EN**: Starts a function, method, lambda, or structured scope: `bool IntrinsicProcTable::IsIntrinsic(const std::string &name) const {`.
  **L4162 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool IntrinsicProcTable::IsIntrinsic(const std::string &name) const {`。
- **L4163 EN**: Returns from the current function with `DEREF(impl_.get()).IsIntrinsic(name)`.
  **L4163 CN**: 以 `DEREF(impl_.get()).IsIntrinsic(name)` 从当前函数返回。
- **L4164 EN**: Closes the current lexical scope or compound statement.
  **L4164 CN**: 结束当前词法作用域或复合语句块。
- **L4165 EN**: Starts a function, method, lambda, or structured scope: `bool IntrinsicProcTable::IsIntrinsicFunction(const std::string &name) const {`.
  **L4165 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool IntrinsicProcTable::IsIntrinsicFunction(const std::string &name) const {`。
- **L4166 EN**: Returns from the current function with `DEREF(impl_.get()).IsIntrinsicFunction(name)`.
  **L4166 CN**: 以 `DEREF(impl_.get()).IsIntrinsicFunction(name)` 从当前函数返回。
- **L4167 EN**: Closes the current lexical scope or compound statement.
  **L4167 CN**: 结束当前词法作用域或复合语句块。
- **L4168 EN**: Starts a function, method, lambda, or structured scope: `bool IntrinsicProcTable::IsIntrinsicSubroutine(const std::string &name) const {`.
  **L4168 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool IntrinsicProcTable::IsIntrinsicSubroutine(const std::string &name) const {`。
- **L4169 EN**: Returns from the current function with `DEREF(impl_.get()).IsIntrinsicSubroutine(name)`.
  **L4169 CN**: 以 `DEREF(impl_.get()).IsIntrinsicSubroutine(name)` 从当前函数返回。
- **L4170 EN**: Closes the current lexical scope or compound statement.
  **L4170 CN**: 结束当前词法作用域或复合语句块。
- **L4171 EN**: Starts a function, method, lambda, or structured scope: `bool IntrinsicProcTable::IsDualIntrinsic(const std::string &name) const {`.
  **L4171 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool IntrinsicProcTable::IsDualIntrinsic(const std::string &name) const {`。
- **L4172 EN**: Returns from the current function with `DEREF(impl_.get()).IsDualIntrinsic(name)`.
  **L4172 CN**: 以 `DEREF(impl_.get()).IsDualIntrinsic(name)` 从当前函数返回。
- **L4173 EN**: Closes the current lexical scope or compound statement.
  **L4173 CN**: 结束当前词法作用域或复合语句块。
- **L4174 EN**: Blank line separating nearby declarations or logic blocks.
  **L4174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4175 EN**: Continues logic associated with callable symbol `GetIntrinsicClass`.
  **L4175 CN**: 继续与可调用符号 `GetIntrinsicClass` 相关的逻辑。
- **L4176 EN**: Continues the surrounding expression or declaration: `const std::string &name) const {`.
  **L4176 CN**: 继续构造周围的表达式或声明：`const std::string &name) const {`。

### Lines 4177-4200

````cpp
  return DEREF(impl_.get()).GetIntrinsicClass(name);
}

std::string IntrinsicProcTable::GetGenericIntrinsicName(
    const std::string &name) const {
  return DEREF(impl_.get()).GetGenericIntrinsicName(name);
}

std::optional<SpecificCall> IntrinsicProcTable::Probe(
    const CallCharacteristics &call, ActualArguments &arguments,
    FoldingContext &context) const {
  return DEREF(impl_.get()).Probe(call, arguments, context);
}

std::optional<SpecificIntrinsicFunctionInterface>
IntrinsicProcTable::IsSpecificIntrinsicFunction(const std::string &name) const {
  return DEREF(impl_.get()).IsSpecificIntrinsicFunction(name);
}

llvm::raw_ostream &TypePattern::Dump(llvm::raw_ostream &o) const {
  if (categorySet == AnyType) {
    o << "any type";
  } else {
    const char *sep = "";
````
- **L4177 EN**: Returns from the current function with `DEREF(impl_.get()).GetIntrinsicClass(name)`.
  **L4177 CN**: 以 `DEREF(impl_.get()).GetIntrinsicClass(name)` 从当前函数返回。
- **L4178 EN**: Closes the current lexical scope or compound statement.
  **L4178 CN**: 结束当前词法作用域或复合语句块。
- **L4179 EN**: Blank line separating nearby declarations or logic blocks.
  **L4179 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4180 EN**: Continues logic associated with callable symbol `GetGenericIntrinsicName`.
  **L4180 CN**: 继续与可调用符号 `GetGenericIntrinsicName` 相关的逻辑。
- **L4181 EN**: Continues the surrounding expression or declaration: `const std::string &name) const {`.
  **L4181 CN**: 继续构造周围的表达式或声明：`const std::string &name) const {`。
- **L4182 EN**: Returns from the current function with `DEREF(impl_.get()).GetGenericIntrinsicName(name)`.
  **L4182 CN**: 以 `DEREF(impl_.get()).GetGenericIntrinsicName(name)` 从当前函数返回。
- **L4183 EN**: Closes the current lexical scope or compound statement.
  **L4183 CN**: 结束当前词法作用域或复合语句块。
- **L4184 EN**: Blank line separating nearby declarations or logic blocks.
  **L4184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4185 EN**: Continues logic associated with callable symbol `Probe`.
  **L4185 CN**: 继续与可调用符号 `Probe` 相关的逻辑。
- **L4186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const CallCharacteristics &call, ActualArguments &arguments,`.
  **L4186 CN**: 继续一个多行参数列表、初始化器或聚合项：`const CallCharacteristics &call, ActualArguments &arguments,`。
- **L4187 EN**: Continues the surrounding expression or declaration: `FoldingContext &context) const {`.
  **L4187 CN**: 继续构造周围的表达式或声明：`FoldingContext &context) const {`。
- **L4188 EN**: Returns from the current function with `DEREF(impl_.get()).Probe(call, arguments, context)`.
  **L4188 CN**: 以 `DEREF(impl_.get()).Probe(call, arguments, context)` 从当前函数返回。
- **L4189 EN**: Closes the current lexical scope or compound statement.
  **L4189 CN**: 结束当前词法作用域或复合语句块。
- **L4190 EN**: Blank line separating nearby declarations or logic blocks.
  **L4190 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4191 EN**: Continues the surrounding expression or declaration: `std::optional<SpecificIntrinsicFunctionInterface>`.
  **L4191 CN**: 继续构造周围的表达式或声明：`std::optional<SpecificIntrinsicFunctionInterface>`。
- **L4192 EN**: Starts a function, method, lambda, or structured scope: `IntrinsicProcTable::IsSpecificIntrinsicFunction(const std::string &name) const {`.
  **L4192 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IntrinsicProcTable::IsSpecificIntrinsicFunction(const std::string &name) const {`。
- **L4193 EN**: Returns from the current function with `DEREF(impl_.get()).IsSpecificIntrinsicFunction(name)`.
  **L4193 CN**: 以 `DEREF(impl_.get()).IsSpecificIntrinsicFunction(name)` 从当前函数返回。
- **L4194 EN**: Closes the current lexical scope or compound statement.
  **L4194 CN**: 结束当前词法作用域或复合语句块。
- **L4195 EN**: Blank line separating nearby declarations or logic blocks.
  **L4195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4196 EN**: Starts a function, method, lambda, or structured scope: `llvm::raw_ostream &TypePattern::Dump(llvm::raw_ostream &o) const {`.
  **L4196 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::raw_ostream &TypePattern::Dump(llvm::raw_ostream &o) const {`。
- **L4197 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4197 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4198 EN**: Executes a standalone statement or declaration: `o << "any type";`.
  **L4198 CN**: 执行一条独立语句或声明：`o << "any type";`。
- **L4199 EN**: Transitions from the previous branch into the alternative path.
  **L4199 CN**: 从前一个分支过渡到备选路径。
- **L4200 EN**: Executes a standalone statement or declaration: `const char *sep = "";`.
  **L4200 CN**: 执行一条独立语句或声明：`const char *sep = "";`。

### Lines 4201-4224

````cpp
    auto set{categorySet};
    while (auto least{set.LeastElement()}) {
      o << sep << EnumToString(*least);
      sep = " or ";
      set.reset(*least);
    }
  }
  o << '(' << EnumToString(kindCode) << ')';
  return o;
}

llvm::raw_ostream &IntrinsicDummyArgument::Dump(llvm::raw_ostream &o) const {
  if (keyword) {
    o << keyword << '=';
  }
  return typePattern.Dump(o)
      << ' ' << EnumToString(rank) << ' ' << EnumToString(optionality)
      << EnumToString(intent);
}

llvm::raw_ostream &IntrinsicInterface::Dump(llvm::raw_ostream &o) const {
  o << name;
  char sep{'('};
  for (const auto &d : dummy) {
````
- **L4201 EN**: Executes a standalone statement or declaration: `auto set{categorySet};`.
  **L4201 CN**: 执行一条独立语句或声明：`auto set{categorySet};`。
- **L4202 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L4202 CN**: 开始 `while` 控制流语句并计算其条件。
- **L4203 EN**: Executes a call or declaration centered on `EnumToString`.
  **L4203 CN**: 执行以 `EnumToString` 为核心的调用或声明。
- **L4204 EN**: Executes a standalone statement or declaration: `sep = " or ";`.
  **L4204 CN**: 执行一条独立语句或声明：`sep = " or ";`。
- **L4205 EN**: Executes a call or declaration centered on `set.reset`.
  **L4205 CN**: 执行以 `set.reset` 为核心的调用或声明。
- **L4206 EN**: Closes the current lexical scope or compound statement.
  **L4206 CN**: 结束当前词法作用域或复合语句块。
- **L4207 EN**: Closes the current lexical scope or compound statement.
  **L4207 CN**: 结束当前词法作用域或复合语句块。
- **L4208 EN**: Executes a call or declaration centered on `'`.
  **L4208 CN**: 执行以 `'` 为核心的调用或声明。
- **L4209 EN**: Returns from the current function with `o`.
  **L4209 CN**: 以 `o` 从当前函数返回。
- **L4210 EN**: Closes the current lexical scope or compound statement.
  **L4210 CN**: 结束当前词法作用域或复合语句块。
- **L4211 EN**: Blank line separating nearby declarations or logic blocks.
  **L4211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4212 EN**: Starts a function, method, lambda, or structured scope: `llvm::raw_ostream &IntrinsicDummyArgument::Dump(llvm::raw_ostream &o) const {`.
  **L4212 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::raw_ostream &IntrinsicDummyArgument::Dump(llvm::raw_ostream &o) const {`。
- **L4213 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4213 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4214 EN**: Executes a standalone statement or declaration: `o << keyword << '=';`.
  **L4214 CN**: 执行一条独立语句或声明：`o << keyword << '=';`。
- **L4215 EN**: Closes the current lexical scope or compound statement.
  **L4215 CN**: 结束当前词法作用域或复合语句块。
- **L4216 EN**: Returns from the current function with `typePattern.Dump(o)`.
  **L4216 CN**: 以 `typePattern.Dump(o)` 从当前函数返回。
- **L4217 EN**: Continues logic associated with callable symbol `EnumToString`.
  **L4217 CN**: 继续与可调用符号 `EnumToString` 相关的逻辑。
- **L4218 EN**: Executes a call or declaration centered on `EnumToString`.
  **L4218 CN**: 执行以 `EnumToString` 为核心的调用或声明。
- **L4219 EN**: Closes the current lexical scope or compound statement.
  **L4219 CN**: 结束当前词法作用域或复合语句块。
- **L4220 EN**: Blank line separating nearby declarations or logic blocks.
  **L4220 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4221 EN**: Starts a function, method, lambda, or structured scope: `llvm::raw_ostream &IntrinsicInterface::Dump(llvm::raw_ostream &o) const {`.
  **L4221 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::raw_ostream &IntrinsicInterface::Dump(llvm::raw_ostream &o) const {`。
- **L4222 EN**: Executes a standalone statement or declaration: `o << name;`.
  **L4222 CN**: 执行一条独立语句或声明：`o << name;`。
- **L4223 EN**: Executes a call or declaration centered on `sep{'`.
  **L4223 CN**: 执行以 `sep{'` 为核心的调用或声明。
- **L4224 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4224 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 4225-4248

````cpp
    if (d.typePattern.kindCode == KindCode::none) {
      break;
    }
    d.Dump(o << sep);
    sep = ',';
  }
  if (sep == '(') {
    o << "()";
  }
  return result.Dump(o << " -> ") << ' ' << EnumToString(rank);
}

llvm::raw_ostream &IntrinsicProcTable::Implementation::Dump(
    llvm::raw_ostream &o) const {
  o << "generic intrinsic functions:\n";
  for (const auto &iter : genericFuncs_) {
    iter.second->Dump(o << iter.first << ": ") << '\n';
  }
  o << "specific intrinsic functions:\n";
  for (const auto &iter : specificFuncs_) {
    iter.second->Dump(o << iter.first << ": ");
    if (const char *g{iter.second->generic}) {
      o << " -> " << g;
    }
````
- **L4225 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4225 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4226 EN**: Exits the nearest loop or switch statement.
  **L4226 CN**: 退出最近的循环或 switch 语句。
- **L4227 EN**: Closes the current lexical scope or compound statement.
  **L4227 CN**: 结束当前词法作用域或复合语句块。
- **L4228 EN**: Executes a call or declaration centered on `d.Dump`.
  **L4228 CN**: 执行以 `d.Dump` 为核心的调用或声明。
- **L4229 EN**: Executes a standalone statement or declaration: `sep = ',';`.
  **L4229 CN**: 执行一条独立语句或声明：`sep = ',';`。
- **L4230 EN**: Closes the current lexical scope or compound statement.
  **L4230 CN**: 结束当前词法作用域或复合语句块。
- **L4231 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4231 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4232 EN**: Executes a call or declaration centered on `"`.
  **L4232 CN**: 执行以 `"` 为核心的调用或声明。
- **L4233 EN**: Closes the current lexical scope or compound statement.
  **L4233 CN**: 结束当前词法作用域或复合语句块。
- **L4234 EN**: Returns from the current function with `result.Dump(o << " -> ") << ' ' << EnumToString(rank)`.
  **L4234 CN**: 以 `result.Dump(o << " -> ") << ' ' << EnumToString(rank)` 从当前函数返回。
- **L4235 EN**: Closes the current lexical scope or compound statement.
  **L4235 CN**: 结束当前词法作用域或复合语句块。
- **L4236 EN**: Blank line separating nearby declarations or logic blocks.
  **L4236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4237 EN**: Continues logic associated with callable symbol `Dump`.
  **L4237 CN**: 继续与可调用符号 `Dump` 相关的逻辑。
- **L4238 EN**: Continues the surrounding expression or declaration: `llvm::raw_ostream &o) const {`.
  **L4238 CN**: 继续构造周围的表达式或声明：`llvm::raw_ostream &o) const {`。
- **L4239 EN**: Executes a standalone statement or declaration: `o << "generic intrinsic functions:\n";`.
  **L4239 CN**: 执行一条独立语句或声明：`o << "generic intrinsic functions:\n";`。
- **L4240 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4240 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4241 EN**: Executes a call or declaration centered on `iter.second->Dump`.
  **L4241 CN**: 执行以 `iter.second->Dump` 为核心的调用或声明。
- **L4242 EN**: Closes the current lexical scope or compound statement.
  **L4242 CN**: 结束当前词法作用域或复合语句块。
- **L4243 EN**: Executes a standalone statement or declaration: `o << "specific intrinsic functions:\n";`.
  **L4243 CN**: 执行一条独立语句或声明：`o << "specific intrinsic functions:\n";`。
- **L4244 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4244 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4245 EN**: Executes a call or declaration centered on `iter.second->Dump`.
  **L4245 CN**: 执行以 `iter.second->Dump` 为核心的调用或声明。
- **L4246 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4246 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4247 EN**: Executes a standalone statement or declaration: `o << " -> " << g;`.
  **L4247 CN**: 执行一条独立语句或声明：`o << " -> " << g;`。
- **L4248 EN**: Closes the current lexical scope or compound statement.
  **L4248 CN**: 结束当前词法作用域或复合语句块。

### Lines 4249-4272

````cpp
    o << '\n';
  }
  o << "subroutines:\n";
  for (const auto &iter : subroutines_) {
    iter.second->Dump(o << iter.first << ": ") << '\n';
  }
  return o;
}

llvm::raw_ostream &IntrinsicProcTable::Dump(llvm::raw_ostream &o) const {
  return DEREF(impl_.get()).Dump(o);
}

// In general C846 prohibits allocatable coarrays to be passed to INTENT(OUT)
// dummy arguments. This rule does not apply to intrinsics in general.
// Some intrinsic explicitly allow coarray allocatable in their description.
// It is assumed that unless explicitly allowed for an intrinsic,
// this is forbidden.
// Since there are very few intrinsic identified that allow this, they are
// listed here instead of adding a field in the table.
bool AcceptsIntentOutAllocatableCoarray(const std::string &intrinsic) {
  return intrinsic == "move_alloc";
}
} // namespace Fortran::evaluate
````
- **L4249 EN**: Executes a standalone statement or declaration: `o << '\n';`.
  **L4249 CN**: 执行一条独立语句或声明：`o << '\n';`。
- **L4250 EN**: Closes the current lexical scope or compound statement.
  **L4250 CN**: 结束当前词法作用域或复合语句块。
- **L4251 EN**: Executes a standalone statement or declaration: `o << "subroutines:\n";`.
  **L4251 CN**: 执行一条独立语句或声明：`o << "subroutines:\n";`。
- **L4252 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4252 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4253 EN**: Executes a call or declaration centered on `iter.second->Dump`.
  **L4253 CN**: 执行以 `iter.second->Dump` 为核心的调用或声明。
- **L4254 EN**: Closes the current lexical scope or compound statement.
  **L4254 CN**: 结束当前词法作用域或复合语句块。
- **L4255 EN**: Returns from the current function with `o`.
  **L4255 CN**: 以 `o` 从当前函数返回。
- **L4256 EN**: Closes the current lexical scope or compound statement.
  **L4256 CN**: 结束当前词法作用域或复合语句块。
- **L4257 EN**: Blank line separating nearby declarations or logic blocks.
  **L4257 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4258 EN**: Starts a function, method, lambda, or structured scope: `llvm::raw_ostream &IntrinsicProcTable::Dump(llvm::raw_ostream &o) const {`.
  **L4258 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::raw_ostream &IntrinsicProcTable::Dump(llvm::raw_ostream &o) const {`。
- **L4259 EN**: Returns from the current function with `DEREF(impl_.get()).Dump(o)`.
  **L4259 CN**: 以 `DEREF(impl_.get()).Dump(o)` 从当前函数返回。
- **L4260 EN**: Closes the current lexical scope or compound statement.
  **L4260 CN**: 结束当前词法作用域或复合语句块。
- **L4261 EN**: Blank line separating nearby declarations or logic blocks.
  **L4261 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4262 EN**: Comment explains nearby logic, intent, or metadata: `In general C846 prohibits allocatable coarrays to be passed to INTENT(OUT)`.
  **L4262 CN**: 注释说明附近代码的逻辑、意图或元数据：`In general C846 prohibits allocatable coarrays to be passed to INTENT(OUT)`。
- **L4263 EN**: Comment explains nearby logic, intent, or metadata: `dummy arguments. This rule does not apply to intrinsics in general.`.
  **L4263 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummy arguments. This rule does not apply to intrinsics in general.`。
- **L4264 EN**: Comment explains nearby logic, intent, or metadata: `Some intrinsic explicitly allow coarray allocatable in their description.`.
  **L4264 CN**: 注释说明附近代码的逻辑、意图或元数据：`Some intrinsic explicitly allow coarray allocatable in their description.`。
- **L4265 EN**: Comment explains nearby logic, intent, or metadata: `It is assumed that unless explicitly allowed for an intrinsic,`.
  **L4265 CN**: 注释说明附近代码的逻辑、意图或元数据：`It is assumed that unless explicitly allowed for an intrinsic,`。
- **L4266 EN**: Comment explains nearby logic, intent, or metadata: `this is forbidden.`.
  **L4266 CN**: 注释说明附近代码的逻辑、意图或元数据：`this is forbidden.`。
- **L4267 EN**: Comment explains nearby logic, intent, or metadata: `Since there are very few intrinsic identified that allow this, they are`.
  **L4267 CN**: 注释说明附近代码的逻辑、意图或元数据：`Since there are very few intrinsic identified that allow this, they are`。
- **L4268 EN**: Comment explains nearby logic, intent, or metadata: `listed here instead of adding a field in the table.`.
  **L4268 CN**: 注释说明附近代码的逻辑、意图或元数据：`listed here instead of adding a field in the table.`。
- **L4269 EN**: Starts a function, method, lambda, or structured scope: `bool AcceptsIntentOutAllocatableCoarray(const std::string &intrinsic) {`.
  **L4269 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool AcceptsIntentOutAllocatableCoarray(const std::string &intrinsic) {`。
- **L4270 EN**: Returns from the current function with `intrinsic == "move_alloc"`.
  **L4270 CN**: 以 `intrinsic == "move_alloc"` 从当前函数返回。
- **L4271 EN**: Closes the current lexical scope or compound statement.
  **L4271 CN**: 结束当前词法作用域或复合语句块。
- **L4272 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::evaluate`.
  **L4272 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::evaluate`。

## Key Concepts / 关键概念

- **Compile-time expression evaluation / 编译期表达式求值**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Constant folding / 常量折叠**
- **Evaluation context management / 求值上下文管理**
- **CUDA-specific lowering or runtime handling / CUDA 专用 lowering 或运行时处理**

## Dependencies / 依赖关系

- `flang/Evaluate/intrinsics.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Common/enum-set.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Common/float128.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Common/idioms.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Evaluate/check-expression.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/common.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/expression.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/fold.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/shape.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/tools.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/type.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Semantics/scope.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Support/Fortran.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `llvm/Support/raw_ostream.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `algorithm`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
