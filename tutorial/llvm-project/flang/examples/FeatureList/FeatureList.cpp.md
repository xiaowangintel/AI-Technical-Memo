# FeatureList.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `flang/examples/FeatureList/FeatureList.cpp`
- Repository: `llvm-project` / `llvm-project`
- Purpose (EN): A plugin that counts the amount of times a particular parse tree node.
- Purpose (CN): 提供围绕 Feature List 的示例或插件代码。

## Line-by-Line Analysis / 逐行分析

### Line 1

~~~~cpp
//===-- FeatureList.cpp ---------------------------------------------------===//
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

~~~~cpp
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 9

~~~~cpp
// A plugin that counts the amount of times a particular parse tree node
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 10

~~~~cpp
// occurs.  This plugin should cover each feature covered in dump-parse-tree.h
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 11

~~~~cpp
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 12

~~~~cpp
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 13

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 14

~~~~cpp
#include "flang/Frontend/FrontendActions.h"
~~~~
- EN: Includes the internal header `flang/Frontend/FrontendActions.h` so this file can use its declarations.
- CN: 引入内部头文件 `flang/Frontend/FrontendActions.h`，以便使用其中的声明。

### Line 15

~~~~cpp
#include "flang/Frontend/FrontendPluginRegistry.h"
~~~~
- EN: Includes the internal header `flang/Frontend/FrontendPluginRegistry.h` so this file can use its declarations.
- CN: 引入内部头文件 `flang/Frontend/FrontendPluginRegistry.h`，以便使用其中的声明。

### Line 16

~~~~cpp
#include "flang/Parser/parse-tree-visitor.h"
~~~~
- EN: Includes the internal header `flang/Parser/parse-tree-visitor.h` so this file can use its declarations.
- CN: 引入内部头文件 `flang/Parser/parse-tree-visitor.h`，以便使用其中的声明。

### Line 17

~~~~cpp
#include "flang/Parser/parse-tree.h"
~~~~
- EN: Includes the internal header `flang/Parser/parse-tree.h` so this file can use its declarations.
- CN: 引入内部头文件 `flang/Parser/parse-tree.h`，以便使用其中的声明。

### Line 18

~~~~cpp
#include "flang/Parser/parsing.h"
~~~~
- EN: Includes the internal header `flang/Parser/parsing.h` so this file can use its declarations.
- CN: 引入内部头文件 `flang/Parser/parsing.h`，以便使用其中的声明。

### Line 19

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 20

~~~~cpp
#include <algorithm>
~~~~
- EN: Includes the external or standard header `<algorithm>` for supporting facilities.
- CN: 引入外部或标准头文件 `<algorithm>` 以获得所需支持功能。

### Line 21

~~~~cpp
#include <cstring>
~~~~
- EN: Includes the external or standard header `<cstring>` for supporting facilities.
- CN: 引入外部或标准头文件 `<cstring>` 以获得所需支持功能。

### Line 22

~~~~cpp
#include <unordered_map>
~~~~
- EN: Includes the external or standard header `<unordered_map>` for supporting facilities.
- CN: 引入外部或标准头文件 `<unordered_map>` 以获得所需支持功能。

### Line 23

~~~~cpp
#include <utility>
~~~~
- EN: Includes the external or standard header `<utility>` for supporting facilities.
- CN: 引入外部或标准头文件 `<utility>` 以获得所需支持功能。

### Line 24

~~~~cpp
#include <vector>
~~~~
- EN: Includes the external or standard header `<vector>` for supporting facilities.
- CN: 引入外部或标准头文件 `<vector>` 以获得所需支持功能。

### Line 25

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 26

~~~~cpp
using namespace Fortran::common;
~~~~
- EN: Imports all names from namespace `Fortran::common` into the current scope.
- CN: 将命名空间 `Fortran::common` 的所有名称导入当前作用域。

### Line 27

~~~~cpp
using namespace Fortran::frontend;
~~~~
- EN: Imports all names from namespace `Fortran::frontend` into the current scope.
- CN: 将命名空间 `Fortran::frontend` 的所有名称导入当前作用域。

### Line 28

~~~~cpp
using namespace Fortran::parser;
~~~~
- EN: Imports all names from namespace `Fortran::parser` into the current scope.
- CN: 将命名空间 `Fortran::parser` 的所有名称导入当前作用域。

### Line 29

~~~~cpp
using namespace Fortran;
~~~~
- EN: Imports all names from namespace `Fortran` into the current scope.
- CN: 将命名空间 `Fortran` 的所有名称导入当前作用域。

### Line 30

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 31

~~~~cpp
#define READ_FEATURE_CUST(classname, n) \
~~~~
- EN: Defines the preprocessor macro `READ_FEATURE_CUST`.
- CN: 定义预处理宏 `READ_FEATURE_CUST`。

### Line 32

~~~~cpp
  bool Pre(const classname &) { \
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 33

~~~~cpp
    record(#n); \
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 34

~~~~cpp
    return true; \
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 35

~~~~cpp
  } \
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 36

~~~~cpp
  void Post(const classname &) {}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 37

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 38

~~~~cpp
#define READ_FEATURE(classname) READ_FEATURE_CUST(classname, classname)
~~~~
- EN: Defines the preprocessor macro `READ_FEATURE`.
- CN: 定义预处理宏 `READ_FEATURE`。

### Line 39

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 40

~~~~cpp
struct NodeVisitor {
~~~~
- EN: Begins the definition of struct `NodeVisitor`.
- CN: 开始定义 struct `NodeVisitor`。

### Line 41

~~~~cpp
private:
~~~~
- EN: Switches subsequent class members to `private` access.
- CN: 将后续类成员切换为 `private` 访问级别。

### Line 42

~~~~cpp
  std::unordered_map<const char *, unsigned int> frequencies;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 43

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 44

~~~~cpp
  void record(const char *name) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 45

~~~~cpp
    auto [it, ins] = frequencies.insert({name, 1});
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 46

~~~~cpp
    if (!ins) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 47

~~~~cpp
      it->second++;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 48

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 49

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 50

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 51

~~~~cpp
public:
~~~~
- EN: Switches subsequent class members to `public` access.
- CN: 将后续类成员切换为 `public` 访问级别。

### Line 52

~~~~cpp
  const std::unordered_map<const char *, unsigned int> &getFrequencies() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 53

~~~~cpp
    return frequencies;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 54

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 55

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 56

~~~~cpp
  READ_FEATURE_CUST(format::ControlEditDesc, ControlEditDesc)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 57

~~~~cpp
  READ_FEATURE_CUST(format::DerivedTypeDataEditDesc, DerivedTypeDataEditDesc)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 58

~~~~cpp
  READ_FEATURE_CUST(format::FormatItem, FormatItem)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 59

~~~~cpp
  READ_FEATURE_CUST(format::FormatSpecification, FormatSpecification)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 60

~~~~cpp
  READ_FEATURE_CUST(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 61

~~~~cpp
      format::IntrinsicTypeDataEditDesc, IntrinsicTypeDataEditDesc)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 62

~~~~cpp
  READ_FEATURE(Abstract)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 63

~~~~cpp
  READ_FEATURE(AccAtomicCapture)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 64

~~~~cpp
  READ_FEATURE(AccAtomicCapture::Stmt1)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 65

~~~~cpp
  READ_FEATURE(AccAtomicCapture::Stmt2)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 66

~~~~cpp
  READ_FEATURE(AccAtomicRead)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 67

~~~~cpp
  READ_FEATURE(AccAtomicUpdate)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 68

~~~~cpp
  READ_FEATURE(AccAtomicWrite)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 69

~~~~cpp
  READ_FEATURE(AccBeginBlockDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 70

~~~~cpp
  READ_FEATURE(AccBeginCombinedDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 71

~~~~cpp
  READ_FEATURE(AccBeginLoopDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 72

~~~~cpp
  READ_FEATURE(AccBlockDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 73

~~~~cpp
  READ_FEATURE(AccClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 74

~~~~cpp
  READ_FEATURE(AccBindClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 75

~~~~cpp
  READ_FEATURE(AccDefaultClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 76

~~~~cpp
  READ_FEATURE(AccClauseList)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 77

~~~~cpp
  READ_FEATURE(AccCombinedDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 78

~~~~cpp
  READ_FEATURE(AccDataModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 79

~~~~cpp
  READ_FEATURE(AccDataModifier::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 80

~~~~cpp
  READ_FEATURE(AccDeclarativeDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 81

~~~~cpp
  READ_FEATURE(AccEndAtomic)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 82

~~~~cpp
  READ_FEATURE(AccEndBlockDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 83

~~~~cpp
  READ_FEATURE(AccEndCombinedDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 84

~~~~cpp
  READ_FEATURE(AccGangArg)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 85

~~~~cpp
  READ_FEATURE(AccObject)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 86

~~~~cpp
  READ_FEATURE(AccObjectList)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 87

~~~~cpp
  READ_FEATURE(AccObjectListWithModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 88

~~~~cpp
  READ_FEATURE(AccObjectListWithReduction)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 89

~~~~cpp
  READ_FEATURE(AccSizeExpr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 90

~~~~cpp
  READ_FEATURE(AccSizeExprList)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 91

~~~~cpp
  READ_FEATURE(AccSelfClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 92

~~~~cpp
  READ_FEATURE(AccStandaloneDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 93

~~~~cpp
  READ_FEATURE(AccDeviceTypeExpr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 94

~~~~cpp
  READ_FEATURE(AccDeviceTypeExprList)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 95

~~~~cpp
  READ_FEATURE(AccTileExpr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 96

~~~~cpp
  READ_FEATURE(AccTileExprList)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 97

~~~~cpp
  READ_FEATURE(AccLoopDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 98

~~~~cpp
  READ_FEATURE(AccWaitArgument)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 99

~~~~cpp
  READ_FEATURE(AcImpliedDo)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 100

~~~~cpp
  READ_FEATURE(AcImpliedDoControl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 101

~~~~cpp
  READ_FEATURE(AcValue)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 102

~~~~cpp
  READ_FEATURE(AccessStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 103

~~~~cpp
  READ_FEATURE(AccessId)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 104

~~~~cpp
  READ_FEATURE(AccessSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 105

~~~~cpp
  READ_FEATURE(AccessSpec::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 106

~~~~cpp
  READ_FEATURE(AcSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 107

~~~~cpp
  READ_FEATURE(ActionStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 108

~~~~cpp
  READ_FEATURE(ActualArg)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 109

~~~~cpp
  READ_FEATURE(ActualArg::PercentRef)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 110

~~~~cpp
  READ_FEATURE(ActualArg::PercentVal)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 111

~~~~cpp
  READ_FEATURE(ActualArgSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 112

~~~~cpp
  READ_FEATURE(AcValue::Triplet)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 113

~~~~cpp
  READ_FEATURE(AllocOpt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 114

~~~~cpp
  READ_FEATURE(AllocOpt::Mold)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 115

~~~~cpp
  READ_FEATURE(AllocOpt::Source)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 116

~~~~cpp
  READ_FEATURE(Allocatable)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 117

~~~~cpp
  READ_FEATURE(AllocatableStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 118

~~~~cpp
  READ_FEATURE(AllocateCoarraySpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 119

~~~~cpp
  READ_FEATURE(AllocateObject)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 120

~~~~cpp
  READ_FEATURE(AllocateShapeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 121

~~~~cpp
  READ_FEATURE(AllocateStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 122

~~~~cpp
  READ_FEATURE(Allocation)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 123

~~~~cpp
  READ_FEATURE(AltReturnSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 124

~~~~cpp
  READ_FEATURE(ArithmeticIfStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 125

~~~~cpp
  READ_FEATURE(ArrayConstructor)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 126

~~~~cpp
  READ_FEATURE(ArrayElement)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 127

~~~~cpp
  READ_FEATURE(ArraySpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 128

~~~~cpp
  READ_FEATURE(AssignStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 129

~~~~cpp
  READ_FEATURE(AssignedGotoStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 130

~~~~cpp
  READ_FEATURE(AssignmentStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 131

~~~~cpp
  READ_FEATURE(AssociateConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 132

~~~~cpp
  READ_FEATURE(AssociateStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 133

~~~~cpp
  READ_FEATURE(Association)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 134

~~~~cpp
  READ_FEATURE(AssumedImpliedSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 135

~~~~cpp
  READ_FEATURE(AssumedRankSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 136

~~~~cpp
  READ_FEATURE(AssumedShapeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 137

~~~~cpp
  READ_FEATURE(AssumedSizeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 138

~~~~cpp
  READ_FEATURE(Asynchronous)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 139

~~~~cpp
  READ_FEATURE(AsynchronousStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 140

~~~~cpp
  READ_FEATURE(AttrSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 141

~~~~cpp
  READ_FEATURE(BOZLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 142

~~~~cpp
  READ_FEATURE(BackspaceStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 143

~~~~cpp
  READ_FEATURE(BasedPointer)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 144

~~~~cpp
  READ_FEATURE(BasedPointerStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 145

~~~~cpp
  READ_FEATURE(BindAttr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 146

~~~~cpp
  READ_FEATURE(BindAttr::Deferred)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 147

~~~~cpp
  READ_FEATURE(BindAttr::Non_Overridable)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 148

~~~~cpp
  READ_FEATURE(BindEntity)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 149

~~~~cpp
  READ_FEATURE(BindEntity::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 150

~~~~cpp
  READ_FEATURE(BindStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 151

~~~~cpp
  READ_FEATURE(Block)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 152

~~~~cpp
  READ_FEATURE(BlockConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 153

~~~~cpp
  READ_FEATURE(BlockData)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 154

~~~~cpp
  READ_FEATURE(BlockDataStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 155

~~~~cpp
  READ_FEATURE(BlockSpecificationPart)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 156

~~~~cpp
  READ_FEATURE(BlockStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 157

~~~~cpp
  READ_FEATURE(BoundsRemapping)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 158

~~~~cpp
  READ_FEATURE(BoundsSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 159

~~~~cpp
  READ_FEATURE(Call)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 160

~~~~cpp
  READ_FEATURE(CallStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 161

~~~~cpp
  READ_FEATURE(CaseConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 162

~~~~cpp
  READ_FEATURE(CaseConstruct::Case)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 163

~~~~cpp
  READ_FEATURE(CaseSelector)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 164

~~~~cpp
  READ_FEATURE(CaseStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 165

~~~~cpp
  READ_FEATURE(CaseValueRange)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 166

~~~~cpp
  READ_FEATURE(CaseValueRange::Range)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 167

~~~~cpp
  READ_FEATURE(ChangeTeamConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 168

~~~~cpp
  READ_FEATURE(ChangeTeamStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 169

~~~~cpp
  READ_FEATURE(CharLength)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 170

~~~~cpp
  READ_FEATURE(CharLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 171

~~~~cpp
  READ_FEATURE(CharLiteralConstantSubstring)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 172

~~~~cpp
  READ_FEATURE(CharSelector)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 173

~~~~cpp
  READ_FEATURE(CharSelector::LengthAndKind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 174

~~~~cpp
  READ_FEATURE(CloseStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 175

~~~~cpp
  READ_FEATURE(CloseStmt::CloseSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 176

~~~~cpp
  READ_FEATURE(CoarrayAssociation)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 177

~~~~cpp
  READ_FEATURE(CoarraySpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 178

~~~~cpp
  READ_FEATURE(CodimensionDecl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 179

~~~~cpp
  READ_FEATURE(CodimensionStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 180

~~~~cpp
  READ_FEATURE(CoindexedNamedObject)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 181

~~~~cpp
  READ_FEATURE(CommonBlockObject)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 182

~~~~cpp
  READ_FEATURE(CommonStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 183

~~~~cpp
  READ_FEATURE(CommonStmt::Block)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 184

~~~~cpp
  READ_FEATURE(CompilerDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 185

~~~~cpp
  READ_FEATURE(CompilerDirective::IgnoreTKR)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 186

~~~~cpp
  READ_FEATURE(CompilerDirective::LoopCount)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 187

~~~~cpp
  READ_FEATURE(CompilerDirective::NameValue)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 188

~~~~cpp
  READ_FEATURE(ComplexLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 189

~~~~cpp
  READ_FEATURE(ComplexPart)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 190

~~~~cpp
  READ_FEATURE(ComponentArraySpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 191

~~~~cpp
  READ_FEATURE(ComponentAttrSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 192

~~~~cpp
  READ_FEATURE(ComponentDataSource)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 193

~~~~cpp
  READ_FEATURE(ComponentDecl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 194

~~~~cpp
  READ_FEATURE(FillDecl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 195

~~~~cpp
  READ_FEATURE(ComponentOrFill)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 196

~~~~cpp
  READ_FEATURE(ComponentDefStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 197

~~~~cpp
  READ_FEATURE(ComponentSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 198

~~~~cpp
  READ_FEATURE(ComputedGotoStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 199

~~~~cpp
  READ_FEATURE(ConcurrentControl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 200

~~~~cpp
  READ_FEATURE(ConcurrentHeader)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 201

~~~~cpp
  READ_FEATURE(ConnectSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 202

~~~~cpp
  READ_FEATURE(ConnectSpec::CharExpr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 203

~~~~cpp
  READ_FEATURE(ConnectSpec::CharExpr::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 204

~~~~cpp
  READ_FEATURE(ConnectSpec::Newunit)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 205

~~~~cpp
  READ_FEATURE(ConnectSpec::Recl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 206

~~~~cpp
  READ_FEATURE(ContainsStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 207

~~~~cpp
  READ_FEATURE(Contiguous)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 208

~~~~cpp
  READ_FEATURE(ContiguousStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 209

~~~~cpp
  READ_FEATURE(ContinueStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 210

~~~~cpp
  READ_FEATURE(CriticalConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 211

~~~~cpp
  READ_FEATURE(CriticalStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 212

~~~~cpp
  READ_FEATURE(CycleStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 213

~~~~cpp
  READ_FEATURE(DataComponentDefStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 214

~~~~cpp
  READ_FEATURE(DataIDoObject)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 215

~~~~cpp
  READ_FEATURE(DataImpliedDo)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 216

~~~~cpp
  READ_FEATURE(DataRef)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 217

~~~~cpp
  READ_FEATURE(DataStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 218

~~~~cpp
  READ_FEATURE(DataStmtConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 219

~~~~cpp
  READ_FEATURE(DataStmtObject)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 220

~~~~cpp
  READ_FEATURE(DataStmtRepeat)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 221

~~~~cpp
  READ_FEATURE(DataStmtSet)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 222

~~~~cpp
  READ_FEATURE(DataStmtValue)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 223

~~~~cpp
  READ_FEATURE(DeallocateStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 224

~~~~cpp
  READ_FEATURE(DeclarationConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 225

~~~~cpp
  READ_FEATURE(DeclarationTypeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 226

~~~~cpp
  READ_FEATURE(DeclarationTypeSpec::Class)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 227

~~~~cpp
  READ_FEATURE(DeclarationTypeSpec::ClassStar)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 228

~~~~cpp
  READ_FEATURE(DeclarationTypeSpec::Record)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 229

~~~~cpp
  READ_FEATURE(DeclarationTypeSpec::Type)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 230

~~~~cpp
  READ_FEATURE(DeclarationTypeSpec::TypeStar)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 231

~~~~cpp
  READ_FEATURE(Default)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 232

~~~~cpp
  READ_FEATURE(DeferredCoshapeSpecList)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 233

~~~~cpp
  READ_FEATURE(DeferredShapeSpecList)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 234

~~~~cpp
  READ_FEATURE(DefinedOpName)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 235

~~~~cpp
  READ_FEATURE(DefinedOperator)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 236

~~~~cpp
  READ_FEATURE(DefinedOperator::IntrinsicOperator)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 237

~~~~cpp
  READ_FEATURE(DerivedTypeDef)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 238

~~~~cpp
  READ_FEATURE(DerivedTypeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 239

~~~~cpp
  READ_FEATURE(DerivedTypeStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 240

~~~~cpp
  READ_FEATURE(Designator)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 241

~~~~cpp
  READ_FEATURE(DimensionStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 242

~~~~cpp
  READ_FEATURE(DimensionStmt::Declaration)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 243

~~~~cpp
  READ_FEATURE(DoConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 244

~~~~cpp
  READ_FEATURE(DummyArg)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 245

~~~~cpp
  READ_FEATURE(ElseIfStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 246

~~~~cpp
  READ_FEATURE(ElseStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 247

~~~~cpp
  READ_FEATURE(ElsewhereStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 248

~~~~cpp
  READ_FEATURE(EndAssociateStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 249

~~~~cpp
  READ_FEATURE(EndBlockDataStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 250

~~~~cpp
  READ_FEATURE(EndBlockStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 251

~~~~cpp
  READ_FEATURE(EndChangeTeamStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 252

~~~~cpp
  READ_FEATURE(EndCriticalStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 253

~~~~cpp
  READ_FEATURE(EndDoStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 254

~~~~cpp
  READ_FEATURE(EndEnumStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 255

~~~~cpp
  READ_FEATURE(EndForallStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 256

~~~~cpp
  READ_FEATURE(EndFunctionStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 257

~~~~cpp
  READ_FEATURE(EndIfStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 258

~~~~cpp
  READ_FEATURE(EndInterfaceStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 259

~~~~cpp
  READ_FEATURE(EndLabel)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 260

~~~~cpp
  READ_FEATURE(EndModuleStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 261

~~~~cpp
  READ_FEATURE(EndMpSubprogramStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 262

~~~~cpp
  READ_FEATURE(EndProgramStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 263

~~~~cpp
  READ_FEATURE(EndSelectStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 264

~~~~cpp
  READ_FEATURE(EndSubmoduleStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 265

~~~~cpp
  READ_FEATURE(EndSubroutineStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 266

~~~~cpp
  READ_FEATURE(EndTypeStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 267

~~~~cpp
  READ_FEATURE(EndWhereStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 268

~~~~cpp
  READ_FEATURE(EndfileStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 269

~~~~cpp
  READ_FEATURE(EntityDecl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 270

~~~~cpp
  READ_FEATURE(EntryStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 271

~~~~cpp
  READ_FEATURE(EnumDef)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 272

~~~~cpp
  READ_FEATURE(EnumDefStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 273

~~~~cpp
  READ_FEATURE(Enumerator)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 274

~~~~cpp
  READ_FEATURE(EnumeratorDefStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 275

~~~~cpp
  READ_FEATURE(EorLabel)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 276

~~~~cpp
  READ_FEATURE(EquivalenceObject)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 277

~~~~cpp
  READ_FEATURE(EquivalenceStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 278

~~~~cpp
  READ_FEATURE(ErrLabel)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 279

~~~~cpp
  READ_FEATURE(ErrorRecovery)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 280

~~~~cpp
  READ_FEATURE(EventPostStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 281

~~~~cpp
  READ_FEATURE(EventWaitStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 282

~~~~cpp
  READ_FEATURE(EventWaitSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 283

~~~~cpp
  READ_FEATURE(ExecutableConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 284

~~~~cpp
  READ_FEATURE(ExecutionPart)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 285

~~~~cpp
  READ_FEATURE(ExecutionPartConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 286

~~~~cpp
  READ_FEATURE(ExitStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 287

~~~~cpp
  READ_FEATURE(ExplicitCoshapeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 288

~~~~cpp
  READ_FEATURE(ExplicitShapeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 289

~~~~cpp
  READ_FEATURE(Expr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 290

~~~~cpp
  READ_FEATURE(Expr::Parentheses)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 291

~~~~cpp
  READ_FEATURE(Expr::UnaryPlus)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 292

~~~~cpp
  READ_FEATURE(Expr::Negate)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 293

~~~~cpp
  READ_FEATURE(Expr::NOT)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 294

~~~~cpp
  READ_FEATURE(Expr::PercentLoc)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 295

~~~~cpp
  READ_FEATURE(Expr::DefinedUnary)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 296

~~~~cpp
  READ_FEATURE(Expr::Power)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 297

~~~~cpp
  READ_FEATURE(Expr::Multiply)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 298

~~~~cpp
  READ_FEATURE(Expr::Divide)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 299

~~~~cpp
  READ_FEATURE(Expr::Add)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 300

~~~~cpp
  READ_FEATURE(Expr::Subtract)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 301

~~~~cpp
  READ_FEATURE(Expr::Concat)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 302

~~~~cpp
  READ_FEATURE(Expr::LT)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 303

~~~~cpp
  READ_FEATURE(Expr::LE)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 304

~~~~cpp
  READ_FEATURE(Expr::EQ)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 305

~~~~cpp
  READ_FEATURE(Expr::NE)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 306

~~~~cpp
  READ_FEATURE(Expr::GE)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 307

~~~~cpp
  READ_FEATURE(Expr::GT)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 308

~~~~cpp
  READ_FEATURE(Expr::AND)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 309

~~~~cpp
  READ_FEATURE(Expr::OR)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 310

~~~~cpp
  READ_FEATURE(Expr::EQV)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 311

~~~~cpp
  READ_FEATURE(Expr::NEQV)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 312

~~~~cpp
  READ_FEATURE(Expr::DefinedBinary)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 313

~~~~cpp
  READ_FEATURE(Expr::ComplexConstructor)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 314

~~~~cpp
  READ_FEATURE(ConditionalExpr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 315

~~~~cpp
  READ_FEATURE(ConditionalArg)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 316

~~~~cpp
  READ_FEATURE(ConditionalArg::Consequent)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 317

~~~~cpp
  READ_FEATURE(ConditionalArgTail)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 318

~~~~cpp
  READ_FEATURE(ConditionalArgNil)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 319

~~~~cpp
  READ_FEATURE(External)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 320

~~~~cpp
  READ_FEATURE(ExternalStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 321

~~~~cpp
  READ_FEATURE(FailImageStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 322

~~~~cpp
  READ_FEATURE(FileUnitNumber)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 323

~~~~cpp
  READ_FEATURE(FinalProcedureStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 324

~~~~cpp
  READ_FEATURE(FlushStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 325

~~~~cpp
  READ_FEATURE(ForallAssignmentStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 326

~~~~cpp
  READ_FEATURE(ForallBodyConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 327

~~~~cpp
  READ_FEATURE(ForallConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 328

~~~~cpp
  READ_FEATURE(ForallConstructStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 329

~~~~cpp
  READ_FEATURE(ForallStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 330

~~~~cpp
  READ_FEATURE(FormTeamStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 331

~~~~cpp
  READ_FEATURE(FormTeamStmt::FormTeamSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 332

~~~~cpp
  READ_FEATURE(Format)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 333

~~~~cpp
  READ_FEATURE(FormatStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 334

~~~~cpp
  READ_FEATURE(FunctionReference)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 335

~~~~cpp
  READ_FEATURE(FunctionStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 336

~~~~cpp
  READ_FEATURE(FunctionSubprogram)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 337

~~~~cpp
  READ_FEATURE(GenericSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 338

~~~~cpp
  READ_FEATURE(GenericSpec::Assignment)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 339

~~~~cpp
  READ_FEATURE(GenericSpec::ReadFormatted)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 340

~~~~cpp
  READ_FEATURE(GenericSpec::ReadUnformatted)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 341

~~~~cpp
  READ_FEATURE(GenericSpec::WriteFormatted)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 342

~~~~cpp
  READ_FEATURE(GenericSpec::WriteUnformatted)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 343

~~~~cpp
  READ_FEATURE(GenericStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 344

~~~~cpp
  READ_FEATURE(GotoStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 345

~~~~cpp
  READ_FEATURE(HollerithLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 346

~~~~cpp
  READ_FEATURE(IdExpr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 347

~~~~cpp
  READ_FEATURE(IdVariable)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 348

~~~~cpp
  READ_FEATURE(IfConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 349

~~~~cpp
  READ_FEATURE(IfConstruct::ElseBlock)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 350

~~~~cpp
  READ_FEATURE(IfConstruct::ElseIfBlock)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 351

~~~~cpp
  READ_FEATURE(IfStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 352

~~~~cpp
  READ_FEATURE(IfThenStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 353

~~~~cpp
  READ_FEATURE(TeamValue)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 354

~~~~cpp
  READ_FEATURE(ImageSelector)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 355

~~~~cpp
  READ_FEATURE(ImageSelectorSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 356

~~~~cpp
  READ_FEATURE(ImageSelectorSpec::Notify)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 357

~~~~cpp
  READ_FEATURE(ImageSelectorSpec::Stat)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 358

~~~~cpp
  READ_FEATURE(ImageSelectorSpec::Team_Number)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 359

~~~~cpp
  READ_FEATURE(ImplicitPart)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 360

~~~~cpp
  READ_FEATURE(ImplicitPartStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 361

~~~~cpp
  READ_FEATURE(ImplicitSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 362

~~~~cpp
  READ_FEATURE(ImplicitStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 363

~~~~cpp
  READ_FEATURE(ImplicitStmt::ImplicitNoneNameSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 364

~~~~cpp
  READ_FEATURE(ImpliedShapeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 365

~~~~cpp
  READ_FEATURE(ImportStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 366

~~~~cpp
  READ_FEATURE(Initialization)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 367

~~~~cpp
  READ_FEATURE(InputImpliedDo)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 368

~~~~cpp
  READ_FEATURE(InputItem)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 369

~~~~cpp
  READ_FEATURE(InquireSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 370

~~~~cpp
  READ_FEATURE(InquireSpec::CharVar)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 371

~~~~cpp
  READ_FEATURE(InquireSpec::CharVar::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 372

~~~~cpp
  READ_FEATURE(InquireSpec::IntVar)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 373

~~~~cpp
  READ_FEATURE(InquireSpec::IntVar::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 374

~~~~cpp
  READ_FEATURE(InquireSpec::LogVar)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 375

~~~~cpp
  READ_FEATURE(InquireSpec::LogVar::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 376

~~~~cpp
  READ_FEATURE(InquireStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 377

~~~~cpp
  READ_FEATURE(InquireStmt::Iolength)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 378

~~~~cpp
  READ_FEATURE(IntegerTypeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 379

~~~~cpp
  READ_FEATURE(IntentSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 380

~~~~cpp
  READ_FEATURE(IntentSpec::Intent)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 381

~~~~cpp
  READ_FEATURE(IntentStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 382

~~~~cpp
  READ_FEATURE(InterfaceBlock)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 383

~~~~cpp
  READ_FEATURE(InterfaceBody)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 384

~~~~cpp
  READ_FEATURE(InterfaceBody::Function)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 385

~~~~cpp
  READ_FEATURE(InterfaceBody::Subroutine)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 386

~~~~cpp
  READ_FEATURE(InterfaceSpecification)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 387

~~~~cpp
  READ_FEATURE(InterfaceStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 388

~~~~cpp
  READ_FEATURE(InternalSubprogram)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 389

~~~~cpp
  READ_FEATURE(InternalSubprogramPart)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 390

~~~~cpp
  READ_FEATURE(Intrinsic)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 391

~~~~cpp
  READ_FEATURE(IntrinsicStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 392

~~~~cpp
  READ_FEATURE(IntrinsicTypeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 393

~~~~cpp
  READ_FEATURE(IntrinsicTypeSpec::Character)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 394

~~~~cpp
  READ_FEATURE(IntrinsicTypeSpec::Complex)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 395

~~~~cpp
  READ_FEATURE(IntrinsicTypeSpec::DoubleComplex)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 396

~~~~cpp
  READ_FEATURE(IntrinsicTypeSpec::DoublePrecision)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 397

~~~~cpp
  READ_FEATURE(IntrinsicTypeSpec::Logical)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 398

~~~~cpp
  READ_FEATURE(IntrinsicTypeSpec::Real)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 399

~~~~cpp
  READ_FEATURE(IoControlSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 400

~~~~cpp
  READ_FEATURE(IoControlSpec::Asynchronous)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 401

~~~~cpp
  READ_FEATURE(IoControlSpec::CharExpr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 402

~~~~cpp
  READ_FEATURE(IoControlSpec::CharExpr::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 403

~~~~cpp
  READ_FEATURE(IoControlSpec::Pos)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 404

~~~~cpp
  READ_FEATURE(IoControlSpec::Rec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 405

~~~~cpp
  READ_FEATURE(IoControlSpec::Size)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 406

~~~~cpp
  READ_FEATURE(IoUnit)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 407

~~~~cpp
  READ_FEATURE(Keyword)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 408

~~~~cpp
  READ_FEATURE(KindParam)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 409

~~~~cpp
  READ_FEATURE(KindSelector)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 410

~~~~cpp
  READ_FEATURE(KindSelector::StarSize)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 411

~~~~cpp
  READ_FEATURE(LabelDoStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 412

~~~~cpp
  READ_FEATURE(LanguageBindingSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 413

~~~~cpp
  READ_FEATURE(LengthSelector)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 414

~~~~cpp
  READ_FEATURE(LetterSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 415

~~~~cpp
  READ_FEATURE(LiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 416

~~~~cpp
  READ_FEATURE(IntLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 417

~~~~cpp
  READ_FEATURE(ReductionOperator)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 418

~~~~cpp
  READ_FEATURE(ReductionOperator::Operator)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 419

~~~~cpp
  READ_FEATURE(LocalitySpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 420

~~~~cpp
  READ_FEATURE(LocalitySpec::DefaultNone)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 421

~~~~cpp
  READ_FEATURE(LocalitySpec::Local)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 422

~~~~cpp
  READ_FEATURE(LocalitySpec::LocalInit)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 423

~~~~cpp
  READ_FEATURE(LocalitySpec::Reduce)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 424

~~~~cpp
  READ_FEATURE(LocalitySpec::Shared)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 425

~~~~cpp
  READ_FEATURE(LockStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 426

~~~~cpp
  READ_FEATURE(LockStmt::LockStat)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 427

~~~~cpp
  READ_FEATURE(LogicalLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 428

~~~~cpp
  READ_FEATURE(LoopControl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 429

~~~~cpp
  READ_FEATURE(LoopControl::Concurrent)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 430

~~~~cpp
  READ_FEATURE(MainProgram)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 431

~~~~cpp
  READ_FEATURE(Map)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 432

~~~~cpp
  READ_FEATURE(Map::EndMapStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 433

~~~~cpp
  READ_FEATURE(Map::MapStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 434

~~~~cpp
  READ_FEATURE(MaskedElsewhereStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 435

~~~~cpp
  READ_FEATURE(Module)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 436

~~~~cpp
  READ_FEATURE(ModuleStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 437

~~~~cpp
  READ_FEATURE(ModuleSubprogram)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 438

~~~~cpp
  READ_FEATURE(ModuleSubprogramPart)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 439

~~~~cpp
  READ_FEATURE(MpSubprogramStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 440

~~~~cpp
  READ_FEATURE(MsgVariable)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 441

~~~~cpp
  READ_FEATURE(Name)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 442

~~~~cpp
  READ_FEATURE(NamedConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 443

~~~~cpp
  READ_FEATURE(NamedConstantDef)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 444

~~~~cpp
  READ_FEATURE(NamelistStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 445

~~~~cpp
  READ_FEATURE(NamelistStmt::Group)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 446

~~~~cpp
  READ_FEATURE(NonLabelDoStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 447

~~~~cpp
  READ_FEATURE(NoPass)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 448

~~~~cpp
  READ_FEATURE(NotifyWaitStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 449

~~~~cpp
  READ_FEATURE(NullifyStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 450

~~~~cpp
  READ_FEATURE(NullInit)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 451

~~~~cpp
  READ_FEATURE(ObjectDecl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 452

~~~~cpp
  READ_FEATURE(OldParameterStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 453

~~~~cpp
  READ_FEATURE(OmpAlignedClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 454

~~~~cpp
  READ_FEATURE(OmpAllocateDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 455

~~~~cpp
  READ_FEATURE(OmpBeginDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 456

~~~~cpp
  READ_FEATURE(OmpBeginSectionsDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 457

~~~~cpp
  READ_FEATURE(OmpBlockConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 458

~~~~cpp
  READ_FEATURE(OmpClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 459

~~~~cpp
  READ_FEATURE(OmpClauseList)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 460

~~~~cpp
  READ_FEATURE(OmpCombinerExpression)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 461

~~~~cpp
  READ_FEATURE(OmpDefaultClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 462

~~~~cpp
  READ_FEATURE(OmpDefaultClause::DataSharingAttribute)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 463

~~~~cpp
  READ_FEATURE(OmpDefaultmapClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 464

~~~~cpp
  READ_FEATURE(OmpDefaultmapClause::ImplicitBehavior)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 465

~~~~cpp
  READ_FEATURE(OmpVariableCategory::Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 466

~~~~cpp
  READ_FEATURE(OmpDependClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 467

~~~~cpp
  READ_FEATURE(OmpDependClause::TaskDep)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 468

~~~~cpp
  READ_FEATURE(OmpDoacross::Sink)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 469

~~~~cpp
  READ_FEATURE(OmpDoacross::Source)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 470

~~~~cpp
  READ_FEATURE(OmpDoacrossClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 471

~~~~cpp
  READ_FEATURE(OmpDependenceType)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 472

~~~~cpp
  READ_FEATURE(OmpDependenceType::Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 473

~~~~cpp
  READ_FEATURE(OmpTaskDependenceType)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 474

~~~~cpp
  READ_FEATURE(OmpTaskDependenceType::Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 475

~~~~cpp
  READ_FEATURE(OmpIteration)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 476

~~~~cpp
  READ_FEATURE(OmpIterationOffset)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 477

~~~~cpp
  READ_FEATURE(OmpIterationVector)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 478

~~~~cpp
  READ_FEATURE(OmpEndDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 479

~~~~cpp
  READ_FEATURE(OmpEndSectionsDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 480

~~~~cpp
  READ_FEATURE(OmpGrainsizeClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 481

~~~~cpp
  READ_FEATURE(OmpGrainsizeClause::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 482

~~~~cpp
  READ_FEATURE(OmpPrescriptiveness)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 483

~~~~cpp
  READ_FEATURE(OmpPrescriptiveness::Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 484

~~~~cpp
  READ_FEATURE(OmpIfClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 485

~~~~cpp
  READ_FEATURE(OmpIfClause::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 486

~~~~cpp
  READ_FEATURE(OmpDirectiveNameModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 487

~~~~cpp
  READ_FEATURE(OmpLinearClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 488

~~~~cpp
  READ_FEATURE(OmpLinearClause::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 489

~~~~cpp
  READ_FEATURE(OmpLinearModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 490

~~~~cpp
  READ_FEATURE(OmpLinearModifier::Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 491

~~~~cpp
  READ_FEATURE(OmpMapClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 492

~~~~cpp
  READ_FEATURE(OmpMapClause::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 493

~~~~cpp
  READ_FEATURE(OmpNumTasksClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 494

~~~~cpp
  READ_FEATURE(OmpNumTasksClause::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 495

~~~~cpp
  READ_FEATURE(OmpObject)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 496

~~~~cpp
  READ_FEATURE(OmpObjectList)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 497

~~~~cpp
  READ_FEATURE(OmpOrderClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 498

~~~~cpp
  READ_FEATURE(OmpOrderClause::Ordering)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 499

~~~~cpp
  READ_FEATURE(OmpOrderModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 500

~~~~cpp
  READ_FEATURE(OmpOrderModifier::Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 501

~~~~cpp
  READ_FEATURE(OmpPreferenceSelector)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 502

~~~~cpp
  READ_FEATURE(OmpPreferenceSpecification)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 503

~~~~cpp
  READ_FEATURE(OmpPreferType)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 504

~~~~cpp
  READ_FEATURE(OmpProcBindClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 505

~~~~cpp
  READ_FEATURE(OmpProcBindClause::AffinityPolicy)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 506

~~~~cpp
  READ_FEATURE(OmpReductionClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 507

~~~~cpp
  READ_FEATURE(OmpInReductionClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 508

~~~~cpp
  READ_FEATURE(OmpInitializerClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 509

~~~~cpp
  READ_FEATURE(OmpReductionIdentifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 510

~~~~cpp
  READ_FEATURE(OmpAllocateClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 511

~~~~cpp
  READ_FEATURE(OmpAllocateClause::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 512

~~~~cpp
  READ_FEATURE(OmpAllocatorSimpleModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 513

~~~~cpp
  READ_FEATURE(OmpAllocatorComplexModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 514

~~~~cpp
  READ_FEATURE(OmpAlignModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 515

~~~~cpp
  READ_FEATURE(OmpScheduleClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 516

~~~~cpp
  READ_FEATURE(OmpScheduleClause::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 517

~~~~cpp
  READ_FEATURE(OmpScheduleClause::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 518

~~~~cpp
  READ_FEATURE(OmpInteropType)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 519

~~~~cpp
  READ_FEATURE(OmpInteropType::Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 520

~~~~cpp
  READ_FEATURE(OmpInitClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 521

~~~~cpp
  READ_FEATURE(OmpInitClause::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 522

~~~~cpp
  READ_FEATURE(OmpUseClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 523

~~~~cpp
  READ_FEATURE(OmpDeviceModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 524

~~~~cpp
  READ_FEATURE(OmpDeviceClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 525

~~~~cpp
  READ_FEATURE(OmpDeviceClause::Modifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 526

~~~~cpp
  READ_FEATURE(OmpDeviceTypeClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 527

~~~~cpp
  READ_FEATURE(OmpDeviceTypeClause::DeviceTypeDescription)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 528

~~~~cpp
  READ_FEATURE(OmpChunkModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 529

~~~~cpp
  READ_FEATURE(OmpChunkModifier::Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 530

~~~~cpp
  READ_FEATURE(OmpOrderingModifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 531

~~~~cpp
  READ_FEATURE(OmpOrderingModifier::Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 532

~~~~cpp
  READ_FEATURE(Only)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 533

~~~~cpp
  READ_FEATURE(OpenACCAtomicConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 534

~~~~cpp
  READ_FEATURE(OpenACCBlockConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 535

~~~~cpp
  READ_FEATURE(OpenACCCacheConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 536

~~~~cpp
  READ_FEATURE(OpenACCCombinedConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 537

~~~~cpp
  READ_FEATURE(OpenACCConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 538

~~~~cpp
  READ_FEATURE(OpenACCDeclarativeConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 539

~~~~cpp
  READ_FEATURE(OpenACCLoopConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 540

~~~~cpp
  READ_FEATURE(OpenMPInteropConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 541

~~~~cpp
  READ_FEATURE(OpenACCRoutineConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 542

~~~~cpp
  READ_FEATURE(OpenACCStandaloneDeclarativeConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 543

~~~~cpp
  READ_FEATURE(OpenACCStandaloneConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 544

~~~~cpp
  READ_FEATURE(OpenACCWaitConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 545

~~~~cpp
  READ_FEATURE(OpenMPAtomicConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 546

~~~~cpp
  READ_FEATURE(OpenMPCancelConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 547

~~~~cpp
  READ_FEATURE(OpenMPCancellationPointConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 548

~~~~cpp
  READ_FEATURE(OpenMPConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 549

~~~~cpp
  READ_FEATURE(OpenMPCriticalConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 550

~~~~cpp
  READ_FEATURE(OpenMPDeclarativeConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 551

~~~~cpp
  READ_FEATURE(OmpDeclareReductionDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 552

~~~~cpp
  READ_FEATURE(OmpDeclareSimdDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 553

~~~~cpp
  READ_FEATURE(OmpDeclareTargetDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 554

~~~~cpp
  READ_FEATURE(OmpMemoryOrderType)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 555

~~~~cpp
  READ_FEATURE(OmpAtomicDefaultMemOrderClause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 556

~~~~cpp
  READ_FEATURE(OpenMPFlushConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 557

~~~~cpp
  READ_FEATURE(OpenMPLoopConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 558

~~~~cpp
  READ_FEATURE(OpenMPAllocatorsConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 559

~~~~cpp
  READ_FEATURE(OmpRequiresDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 560

~~~~cpp
  READ_FEATURE(OpenMPSimpleStandaloneConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 561

~~~~cpp
  READ_FEATURE(OpenMPStandaloneConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 562

~~~~cpp
  READ_FEATURE(OmpSectionDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 563

~~~~cpp
  READ_FEATURE(OpenMPSectionsConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 564

~~~~cpp
  READ_FEATURE(OmpThreadprivateDirective)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 565

~~~~cpp
  READ_FEATURE(OpenStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 566

~~~~cpp
  READ_FEATURE(Optional)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 567

~~~~cpp
  READ_FEATURE(OptionalStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 568

~~~~cpp
  READ_FEATURE(OtherSpecificationStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 569

~~~~cpp
  READ_FEATURE(OutputImpliedDo)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 570

~~~~cpp
  READ_FEATURE(OutputItem)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 571

~~~~cpp
  READ_FEATURE(Parameter)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 572

~~~~cpp
  READ_FEATURE(ParameterStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 573

~~~~cpp
  READ_FEATURE(ParentIdentifier)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 574

~~~~cpp
  READ_FEATURE(Pass)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 575

~~~~cpp
  READ_FEATURE(PauseStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 576

~~~~cpp
  READ_FEATURE(Pointer)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 577

~~~~cpp
  READ_FEATURE(PointerAssignmentStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 578

~~~~cpp
  READ_FEATURE(PointerAssignmentStmt::Bounds)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 579

~~~~cpp
  READ_FEATURE(PointerDecl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 580

~~~~cpp
  READ_FEATURE(PointerObject)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 581

~~~~cpp
  READ_FEATURE(PointerStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 582

~~~~cpp
  READ_FEATURE(PositionOrFlushSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 583

~~~~cpp
  READ_FEATURE(PrefixSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 584

~~~~cpp
  READ_FEATURE(PrefixSpec::Elemental)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 585

~~~~cpp
  READ_FEATURE(PrefixSpec::Impure)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 586

~~~~cpp
  READ_FEATURE(PrefixSpec::Module)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 587

~~~~cpp
  READ_FEATURE(PrefixSpec::Non_Recursive)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 588

~~~~cpp
  READ_FEATURE(PrefixSpec::Pure)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 589

~~~~cpp
  READ_FEATURE(PrefixSpec::Recursive)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 590

~~~~cpp
  READ_FEATURE(PrintStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 591

~~~~cpp
  READ_FEATURE(PrivateStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 592

~~~~cpp
  READ_FEATURE(PrivateOrSequence)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 593

~~~~cpp
  READ_FEATURE(ProcAttrSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 594

~~~~cpp
  READ_FEATURE(ProcComponentAttrSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 595

~~~~cpp
  READ_FEATURE(ProcComponentDefStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 596

~~~~cpp
  READ_FEATURE(ProcComponentRef)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 597

~~~~cpp
  READ_FEATURE(ProcDecl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 598

~~~~cpp
  READ_FEATURE(ProcInterface)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 599

~~~~cpp
  READ_FEATURE(ProcPointerInit)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 600

~~~~cpp
  READ_FEATURE(ProcedureDeclarationStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 601

~~~~cpp
  READ_FEATURE(ProcedureDesignator)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 602

~~~~cpp
  READ_FEATURE(ProcedureStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 603

~~~~cpp
  READ_FEATURE(ProcedureStmt::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 604

~~~~cpp
  READ_FEATURE(Program)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 605

~~~~cpp
  READ_FEATURE(ProgramStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 606

~~~~cpp
  READ_FEATURE(ProgramUnit)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 607

~~~~cpp
  READ_FEATURE(Protected)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 608

~~~~cpp
  READ_FEATURE(ProtectedStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 609

~~~~cpp
  READ_FEATURE(ReadStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 610

~~~~cpp
  READ_FEATURE(RealLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 611

~~~~cpp
  READ_FEATURE(RealLiteralConstant::Real)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 612

~~~~cpp
  READ_FEATURE(Rename)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 613

~~~~cpp
  READ_FEATURE(Rename::Names)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 614

~~~~cpp
  READ_FEATURE(Rename::Operators)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 615

~~~~cpp
  READ_FEATURE(ReturnStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 616

~~~~cpp
  READ_FEATURE(RewindStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 617

~~~~cpp
  READ_FEATURE(Save)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 618

~~~~cpp
  READ_FEATURE(SaveStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 619

~~~~cpp
  READ_FEATURE(SavedEntity)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 620

~~~~cpp
  READ_FEATURE(SavedEntity::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 621

~~~~cpp
  READ_FEATURE(SectionSubscript)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 622

~~~~cpp
  READ_FEATURE(SelectCaseStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 623

~~~~cpp
  READ_FEATURE(SelectRankCaseStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 624

~~~~cpp
  READ_FEATURE(SelectRankCaseStmt::Rank)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 625

~~~~cpp
  READ_FEATURE(SelectRankConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 626

~~~~cpp
  READ_FEATURE(SelectRankConstruct::RankCase)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 627

~~~~cpp
  READ_FEATURE(SelectRankStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 628

~~~~cpp
  READ_FEATURE(SelectTypeConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 629

~~~~cpp
  READ_FEATURE(SelectTypeConstruct::TypeCase)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 630

~~~~cpp
  READ_FEATURE(SelectTypeStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 631

~~~~cpp
  READ_FEATURE(Selector)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 632

~~~~cpp
  READ_FEATURE(SeparateModuleSubprogram)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 633

~~~~cpp
  READ_FEATURE(SequenceStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 634

~~~~cpp
  READ_FEATURE(Sign)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 635

~~~~cpp
  READ_FEATURE(SignedComplexLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 636

~~~~cpp
  READ_FEATURE(SignedIntLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 637

~~~~cpp
  READ_FEATURE(SignedRealLiteralConstant)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 638

~~~~cpp
  READ_FEATURE(SpecificationConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 639

~~~~cpp
  READ_FEATURE(SpecificationExpr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 640

~~~~cpp
  READ_FEATURE(SpecificationPart)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 641

~~~~cpp
  READ_FEATURE(Star)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 642

~~~~cpp
  READ_FEATURE(StatOrErrmsg)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 643

~~~~cpp
  READ_FEATURE(StatVariable)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 644

~~~~cpp
  READ_FEATURE(StatusExpr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 645

~~~~cpp
  READ_FEATURE(StmtFunctionStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 646

~~~~cpp
  READ_FEATURE(StopCode)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 647

~~~~cpp
  READ_FEATURE(StopStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 648

~~~~cpp
  READ_FEATURE(StopStmt::Kind)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 649

~~~~cpp
  READ_FEATURE(StructureComponent)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 650

~~~~cpp
  READ_FEATURE(StructureConstructor)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 651

~~~~cpp
  READ_FEATURE(StructureDef)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 652

~~~~cpp
  READ_FEATURE(StructureDef::EndStructureStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 653

~~~~cpp
  READ_FEATURE(StructureField)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 654

~~~~cpp
  READ_FEATURE(StructureStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 655

~~~~cpp
  READ_FEATURE(Submodule)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 656

~~~~cpp
  READ_FEATURE(SubmoduleStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 657

~~~~cpp
  READ_FEATURE(SubroutineStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 658

~~~~cpp
  READ_FEATURE(SubroutineSubprogram)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 659

~~~~cpp
  READ_FEATURE(SubscriptTriplet)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 660

~~~~cpp
  READ_FEATURE(Substring)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 661

~~~~cpp
  READ_FEATURE(SubstringInquiry)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 662

~~~~cpp
  READ_FEATURE(SubstringRange)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 663

~~~~cpp
  READ_FEATURE(Suffix)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 664

~~~~cpp
  READ_FEATURE(SyncAllStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 665

~~~~cpp
  READ_FEATURE(SyncImagesStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 666

~~~~cpp
  READ_FEATURE(SyncImagesStmt::ImageSet)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 667

~~~~cpp
  READ_FEATURE(SyncMemoryStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 668

~~~~cpp
  READ_FEATURE(SyncTeamStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 669

~~~~cpp
  READ_FEATURE(Target)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 670

~~~~cpp
  READ_FEATURE(TargetStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 671

~~~~cpp
  READ_FEATURE(TypeAttrSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 672

~~~~cpp
  READ_FEATURE(TypeAttrSpec::BindC)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 673

~~~~cpp
  READ_FEATURE(TypeAttrSpec::Extends)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 674

~~~~cpp
  READ_FEATURE(TypeBoundGenericStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 675

~~~~cpp
  READ_FEATURE(TypeBoundProcBinding)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 676

~~~~cpp
  READ_FEATURE(TypeBoundProcDecl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 677

~~~~cpp
  READ_FEATURE(TypeBoundProcedurePart)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 678

~~~~cpp
  READ_FEATURE(TypeBoundProcedureStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 679

~~~~cpp
  READ_FEATURE(TypeBoundProcedureStmt::WithInterface)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 680

~~~~cpp
  READ_FEATURE(TypeBoundProcedureStmt::WithoutInterface)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 681

~~~~cpp
  READ_FEATURE(TypeDeclarationStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 682

~~~~cpp
  READ_FEATURE(TypeGuardStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 683

~~~~cpp
  READ_FEATURE(TypeGuardStmt::Guard)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 684

~~~~cpp
  READ_FEATURE(TypeParamDecl)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 685

~~~~cpp
  READ_FEATURE(TypeParamDefStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 686

~~~~cpp
  READ_FEATURE(common::TypeParamAttr)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 687

~~~~cpp
  READ_FEATURE(TypeParamSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 688

~~~~cpp
  READ_FEATURE(TypeParamValue)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 689

~~~~cpp
  READ_FEATURE(TypeParamValue::Deferred)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 690

~~~~cpp
  READ_FEATURE(TypeSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 691

~~~~cpp
  READ_FEATURE(Union)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 692

~~~~cpp
  READ_FEATURE(Union::EndUnionStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 693

~~~~cpp
  READ_FEATURE(Union::UnionStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 694

~~~~cpp
  READ_FEATURE(UnlockStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 695

~~~~cpp
  READ_FEATURE(UseStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 696

~~~~cpp
  READ_FEATURE(UseStmt::ModuleNature)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 697

~~~~cpp
  READ_FEATURE(Value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 698

~~~~cpp
  READ_FEATURE(ValueStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 699

~~~~cpp
  READ_FEATURE(Variable)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 700

~~~~cpp
  READ_FEATURE(Verbatim)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 701

~~~~cpp
  READ_FEATURE(Volatile)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 702

~~~~cpp
  READ_FEATURE(VolatileStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 703

~~~~cpp
  READ_FEATURE(WaitSpec)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 704

~~~~cpp
  READ_FEATURE(WaitStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 705

~~~~cpp
  READ_FEATURE(WhereBodyConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 706

~~~~cpp
  READ_FEATURE(WhereConstruct)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 707

~~~~cpp
  READ_FEATURE(WhereConstruct::Elsewhere)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 708

~~~~cpp
  READ_FEATURE(WhereConstruct::MaskedElsewhere)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 709

~~~~cpp
  READ_FEATURE(WhereConstructStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 710

~~~~cpp
  READ_FEATURE(WhereStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 711

~~~~cpp
  READ_FEATURE(WriteStmt)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 712

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 713

~~~~cpp
  READ_FEATURE(llvm::omp::Directive)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 714

~~~~cpp
  READ_FEATURE(llvm::omp::Clause)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 715

~~~~cpp
  READ_FEATURE(llvm::acc::Directive)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 716

~~~~cpp
  READ_FEATURE(llvm::acc::DefaultValue)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 717

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 718

~~~~cpp
  template <typename A> bool Pre(const A &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 719

~~~~cpp
  template <typename A> void Post(const A &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 720

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 721

~~~~cpp
  template <typename T> bool Pre(const Statement<T> &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 722

~~~~cpp
  template <typename T> void Post(const Statement<T> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 723

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 724

~~~~cpp
  template <typename T> bool Pre(const UnlabeledStatement<T> &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 725

~~~~cpp
  template <typename T> void Post(const UnlabeledStatement<T> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 726

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 727

~~~~cpp
  template <typename T> bool Pre(const common::Indirection<T> &) {
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 728

~~~~cpp
    return true;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 729

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 730

~~~~cpp
  template <typename T> void Post(const common::Indirection<T> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 731

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 732

~~~~cpp
  template <typename A> bool Pre(const Scalar<A> &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 733

~~~~cpp
  template <typename A> void Post(const Scalar<A> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 734

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 735

~~~~cpp
  template <typename A> bool Pre(const Constant<A> &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 736

~~~~cpp
  template <typename A> void Post(const Constant<A> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 737

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 738

~~~~cpp
  template <typename A> bool Pre(const Integer<A> &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 739

~~~~cpp
  template <typename A> void Post(const Integer<A> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 740

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 741

~~~~cpp
  template <typename A> bool Pre(const Logical<A> &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 742

~~~~cpp
  template <typename A> void Post(const Logical<A> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 743

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 744

~~~~cpp
  template <typename A> bool Pre(const DefaultChar<A> &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 745

~~~~cpp
  template <typename A> void Post(const DefaultChar<A> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 746

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 747

~~~~cpp
  template <typename... A> bool Pre(const std::tuple<A...> &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 748

~~~~cpp
  template <typename... A> void Post(const std::tuple<A...> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 749

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 750

~~~~cpp
  template <typename... A> bool Pre(const std::variant<A...> &) { return true; }
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 751

~~~~cpp
  template <typename... A> void Post(const std::variant<A...> &) {}
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 752

~~~~cpp
};
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 753

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 754

~~~~cpp
bool sortNodes(std::pair<const char *, int> a, std::pair<const char *, int> b) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 755

~~~~cpp
  return (a.second == b.second) ? (std::strcmp(a.first, b.first) < 0)
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 756

~~~~cpp
                                : a.second > b.second;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 757

~~~~cpp
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 758

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 759

~~~~cpp
class FeatureListAction : public PluginParseTreeAction {
~~~~
- EN: Begins the definition of class `FeatureListAction`.
- CN: 开始定义 class `FeatureListAction`。

### Line 760

~~~~cpp
  void executeAction() override {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 761

~~~~cpp
    NodeVisitor visitor;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 762

~~~~cpp
    Fortran::parser::Walk(getParsing().parseTree(), visitor);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 763

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 764

~~~~cpp
    const auto &frequencyMap = visitor.getFrequencies();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 765

~~~~cpp
    std::vector<std::pair<const char *, int>> frequencies(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 766

~~~~cpp
        frequencyMap.begin(), frequencyMap.end());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 767

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 768

~~~~cpp
    std::sort(frequencies.begin(), frequencies.end(), sortNodes);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 769

~~~~cpp
    for (auto const &[feature, frequency] : frequencies) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 770

~~~~cpp
      llvm::outs() << feature << ": " << frequency << "\n";
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 771

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 772

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 773

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 774

~~~~cpp
  bool beginSourceFileAction() override {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 775

~~~~cpp
    return runPrescan() && runParse(/*emitMessages=*/true);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 776

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 777

~~~~cpp
};
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 778

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 779

~~~~cpp
static FrontendPluginRegistry::Add<FeatureListAction> X(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 780

~~~~cpp
    "feature-list", "List program features");
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

## Key Concepts / 关键概念

- **Example-oriented implementation / 面向示例的实现**: The file demonstrates a focused Flang extension point or usage pattern. / 该文件展示了一个聚焦的 Flang 扩展点或使用模式。
- **Templates and generic programming / 模板与泛型编程**: Templates let the file abstract behavior across types or compile-time parameters. / 模板让该文件能够在类型或编译期参数之间抽象行为。
- **Namespace organization / 命名空间组织**: Namespaces keep related symbols grouped and reduce accidental collisions. / 命名空间用于组织相关符号并减少命名冲突。
- **Preprocessor coordination / 预处理器协作**: Preprocessor directives shape how the file is configured or expanded. / 预处理指令决定文件的配置方式或展开方式。
- **Type modeling / 类型建模**: Classes, structs, or enums capture domain concepts in a typed form. / 类、结构体或枚举以类型化方式表达领域概念。

## Dependencies / 依赖关系

- **Internal project dependencies / 工程内依赖**:
  - `flang/Frontend/FrontendActions.h` — referenced directly from this file / 该文件直接引用
  - `flang/Frontend/FrontendPluginRegistry.h` — referenced directly from this file / 该文件直接引用
  - `flang/Parser/parse-tree-visitor.h` — referenced directly from this file / 该文件直接引用
  - `flang/Parser/parse-tree.h` — referenced directly from this file / 该文件直接引用
  - `flang/Parser/parsing.h` — referenced directly from this file / 该文件直接引用
- **External or standard dependencies / 外部或标准依赖**:
  - `<algorithm>` — supporting library header / 支撑性库头文件
  - `<cstring>` — supporting library header / 支撑性库头文件
  - `<unordered_map>` — supporting library header / 支撑性库头文件
  - `<utility>` — supporting library header / 支撑性库头文件
  - `<vector>` — supporting library header / 支撑性库头文件
