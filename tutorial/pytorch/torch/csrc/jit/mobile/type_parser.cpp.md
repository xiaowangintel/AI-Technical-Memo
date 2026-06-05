# type_parser.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/mobile/type_parser.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements loading, execution, optimization, or serialization support for the mobile / lite interpreter runtime. This specific file centers on `type_parser.cpp`. It is tailored for lite interpreter or mobile deployment constraints.
- **Purpose (CN)**: 实现移动端 / 轻量解释器运行时的加载、执行、优化或序列化支持。 该文件具体围绕 `type_parser.cpp` 展开。 其设计面向轻量解释器或移动端部署约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <torch/csrc/jit/mobile/type_parser.h>

#include <ATen/core/jit_type.h>
#include <ATen/core/type_factory.h>
#include <torch/csrc/jit/frontend/parser_constants.h>
#include <torch/custom_class.h>
#include <string_view>

using torch::jit::valid_single_char_tokens;

namespace c10 {

namespace {

// Torchbind custom class always starts with the follow prefix, so use it as
// an identifier for torchbind custom class type
static constexpr const char* kTypeTorchbindCustomClass =
    "__torch__.torch.classes";
static constexpr const char* kTypeNamedTuple = "NamedTuple";

bool isSpecialChar(char a) {
  return std::strchr(valid_single_char_tokens, a);
}
} // namespace

TypeParser::TypeParser(std::string pythonStr)
    : pythonStr_(std::move(pythonStr)), start_(0) {
  lex();
```

- **EN:** It enters or references namespace scopes such as c10, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 c10 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** The block declares or refines core types including always, type.
- **CN:** 该代码块声明或细化了 always, type 等核心类型。
- **EN:** Important callable entry points in this range include isSpecialChar, strchr, TypeParser, lex.
- **CN:** 这一段的重要可调用入口包括 isSpecialChar, strchr, TypeParser, lex。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。

### Lines 29-56 / 第 29-56 行

```cpp
}

TypeParser::TypeParser(std::vector<std::string>& pythonStrs)
    : start_(0), pythonStrs_(pythonStrs) {}

// For the Python string list parsing, the order of the Python string matters.
// In bytecode, the order of the type list correspondings to the order of
// instruction. In nested type, the lowest level type will be at the beginning
// of the type list. It is possible to parse it without worrying about
// ordering, but it also introduces 1) extra cost to process nested type to
// the correct order 2) lost the benefit that the instruction order is likely
// problematic if type list parsing fails.
std::vector<TypePtr> TypeParser::parseList() {
  std::vector<TypePtr> typePtrs;
  typePtrs.resize(pythonStrs_.size());
  static const c10::QualifiedName classPrefix = "__torch__.torch.classes";
  for (size_t i = 0; i < pythonStrs_.size(); i++) {
    c10::QualifiedName qn(pythonStrs_[i]);
    c10::TypePtr type_ptr;
    if (classPrefix.isPrefixOf(qn)) {
      type_ptr = torch::getCustomClass(qn.qualifiedName());
      TORCH_CHECK(
          type_ptr,
          "The implementation of class ",
          qn.qualifiedName(),
          " cannot be found.");
    } else {
      pythonStr_ = pythonStrs_[i];
```

- **EN:** Important callable entry points in this range include TypeParser, parseList, qn, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 TypeParser, parseList, qn, TORCH_CHECK。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 57-84 / 第 57-84 行

```cpp
      start_ = 0;
      lex();
      type_ptr = parse();
    }
    typePtrs[i] = type_ptr;
    str_type_ptr_map_[type_ptr->repr_str()] = type_ptr;
  }
  return typePtrs;
}

// The list of non-simple types supported by current parser.
const std::unordered_set<std::string>& TypeParser::getNonSimpleType() {
  static std::unordered_set<std::string> nonSimpleTypes{
      "List", "Optional", "Dict", "Tuple"};
  return nonSimpleTypes;
}

// The list of custom types supported by current parser.
const std::unordered_set<std::string>& TypeParser::getCustomType() {
  static std::unordered_set<std::string> customeTypes{
      kTypeTorchbindCustomClass, kTypeNamedTuple};
  return customeTypes;
}

// Given a PyThon str, get all contained types. It's usually used for
// compatibility check between model and runtime. For example:
// PyThon string: "Dict[int, Tuple[Tensor, Tensor, Tensor]]"
// contained type is: [Dict, int, Tuple, Tensor]
```

- **EN:** Important callable entry points in this range include lex, getNonSimpleType, getCustomType.
- **CN:** 这一段的重要可调用入口包括 lex, getNonSimpleType, getCustomType。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Result propagation / 结果传递。

### Lines 85-112 / 第 85-112 行

```cpp
std::unordered_set<std::string> TypeParser::getContainedTypes() {
  return contained_types_;
}

template <typename T>
TypePtr TypeParser::parseSingleElementType() {
  expectChar('[');
  auto result = DynamicTypeFactory::create<T>(parse());
  expectChar(']');
  return result;
}

TypePtr TypeParser::parseNonSimple(const std::string& token) {
  if (token == "List") {
    return parseSingleElementType<ListType>();
  } else if (token == "Optional") {
    return parseSingleElementType<OptionalType>();
  } else if (token == "Dict") {
    expectChar('[');
    auto key = parse();
    expectChar(',');
    auto val = parse();
    expectChar(']');
    return DynamicTypeFactory::create<DictType>(std::move(key), std::move(val));
  } else if (token == "Tuple") {
    std::vector<TypePtr> types;
    expectChar('[');
    while (cur() != "]") {
```

- **EN:** Important callable entry points in this range include getContainedTypes, parseSingleElementType, expectChar, parseNonSimple.
- **CN:** 这一段的重要可调用入口包括 getContainedTypes, parseSingleElementType, expectChar, parseNonSimple。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 113-140 / 第 113-140 行

```cpp
      types.emplace_back(parse());
      if (cur() != "]") {
        expectChar(',');
      }
    }
    expect("]");
    return DynamicTypeFactory::create<TupleType>(types);
  }
  return nullptr;
}

TypePtr TypeParser::parse() {
  std::string token = next();
  const auto& baseTypes = DynamicTypeFactory::basePythonTypes();
  auto simpleTypeIt = baseTypes.find(token);
  if (simpleTypeIt != baseTypes.end()) {
    if (cur() != "]" && cur() != "," && !cur().empty()) {
      TORCH_CHECK(
          false, "Simple type ", token, " is followed by ", "invalid chars.");
    }
    contained_types_.insert(token);
    return simpleTypeIt->second;
  } else if (getNonSimpleType().find(token) != getNonSimpleType().end()) {
    contained_types_.insert(token);
    return parseNonSimple(token);
  } else if (token == "__torch__") {
    expectChar('.');
    if (cur() == "torch") {
```

- **EN:** Important callable entry points in this range include expectChar, expect, parse, TORCH_CHECK, parseNonSimple.
- **CN:** 这一段的重要可调用入口包括 expectChar, expect, parse, TORCH_CHECK, parseNonSimple。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Type system / 类型系统, Operator schema / 算子模式, Declared symbols / 声明的符号, Result propagation / 结果传递, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Operator schema / 算子模式, Declared symbols / 声明的符号, Result propagation / 结果传递, Branching logic / 分支逻辑。

### Lines 141-168 / 第 141-168 行

```cpp
      // torch bind class starts with __torch__.torch.classes
      return parseTorchbindClassType();
    } else {
      // other class starts with __torch__ following by custom names
      return parseCustomType();
    }
  } else if (token == "Union") {
    // TODO Union types are not supported on embedded runtime, and we need to
    // generate compiler errors for users scripting UnionTypes. Right now
    // for preserving backward compatibility we have to return a nullptr since
    // it does not get involved in type reflection.
    return nullptr;
  } else {
    TORCH_CHECK(
        false,
        "Type ",
        token,
        " is not supported in the parser, ",
        "or the token is in wrong format.");
  }
}

// NamedTuple custom type will be following structure:
// "qualified_named[
//   NamedTuple, [
//       [filed_name_1, field_type_1],
//       [filed_name_2, field_type_2]
//   ]
```

- **EN:** The block declares or refines core types including starts.
- **CN:** 该代码块声明或细化了 starts 等核心类型。
- **EN:** Important callable entry points in this range include parseTorchbindClassType, parseCustomType, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 parseTorchbindClassType, parseCustomType, TORCH_CHECK。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 169-196 / 第 169-196 行

```cpp
// ]"
//  Example NamedTuple type:
//  "__torch__.base_models.sparse_nn.pytorch_preproc_types.PreprocOutputType[
//     NamedTuple, [
//         [float_features, Tensor],
//         [id_list_features, List[Tensor]],
//         [label,  Tensor],
//         [weight, Tensor],
//         ]
//     ]"
TypePtr TypeParser::parseNamedTuple(const std::string& qualified_name) {
  std::vector<std::string_view> field_names;
  std::vector<TypePtr> field_types;
  expect(",");
  expect("[");
  while (cur() != "]") {
    expect("[");
    auto field_name = nextView();
    expect(",");
    TypePtr field_type = parse();
    field_names.emplace_back(field_name);
    field_types.emplace_back(field_type);
    expect("]");
    if (cur() == ",") {
      next();
    }
  }
  return DynamicTypeFactory::createNamedTuple(
```

- **EN:** Important callable entry points in this range include parseNamedTuple, expect, next.
- **CN:** 这一段的重要可调用入口包括 parseNamedTuple, expect, next。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 197-224 / 第 197-224 行

```cpp
      qualified_name, field_names, field_types);
}

// Custom type will be following structure:
// "qualified_named[
//   custom_type, [
//       [filed_name_1, field_type_1],
//       [filed_name_2, field_type_2]
//   ]
// ]"
TypePtr TypeParser::parseCustomType() {
  std::string_view token = cur();
  std::string qualified_name = "__torch__.";
  qualified_name.reserve(qualified_name.size() + token.size());
  qualified_name.append(token.begin(), token.end());
  next();
  while (cur() == ".") {
    qualified_name.append(next());
    qualified_name.append(next());
  }
  // After cur() moves to the next token after qualified name, if it's "[", it
  // means this custom type follow by it's class definition. Otherwise, it's a
  // barebone qualified name and needs to look up str_type_ptr_map_ to find
  // the typeptr.
  if (cur() == "[") {
    next();
    std::string type_name = next();
    // Currently only supports NamedTuple custom type, if more types need to
```

- **EN:** The block declares or refines core types including definition.
- **CN:** 该代码块声明或细化了 definition 等核心类型。
- **EN:** Important callable entry points in this range include parseCustomType, next.
- **CN:** 这一段的重要可调用入口包括 parseCustomType, next。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 225-252 / 第 225-252 行

```cpp
    // be supported, extend them here.
    if (type_name == kTypeNamedTuple) {
      contained_types_.insert(kTypeNamedTuple);
      return parseNamedTuple(qualified_name);
    } else {
      TORCH_CHECK(
          false, "Custom Type ", type_name, " is not supported in the parser.");
    }
  } else {
    auto find_type = str_type_ptr_map_.find(qualified_name);
    if (find_type != str_type_ptr_map_.end()) {
      return find_type->second;
    } else {
      // When the type definition can't be found, likely two reasons
      // 1. The type list in bytecode.pkl is not in the correct order
      // 2. This custom type definition doesn't exist in bytecode.pkl type
      // table
      TORCH_CHECK(
          false, "Can't find definition for the type: ", qualified_name);
    }
  }
}

TypePtr TypeParser::parseTorchbindClassType() {
  static constexpr std::array<const char*, 4> expected_atoms = {
      "torch", ".", "classes", "."};
  for (const auto& atom : expected_atoms) {
    expect(atom);
```

- **EN:** Important callable entry points in this range include parseNamedTuple, TORCH_CHECK, parseTorchbindClassType, expect.
- **CN:** 这一段的重要可调用入口包括 parseNamedTuple, TORCH_CHECK, parseTorchbindClassType, expect。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 253-280 / 第 253-280 行

```cpp
  }
  std::string ns = next();
  expectChar('.');
  std::string classname = next();
  std::string customClassName = "__torch__.torch.classes.";
  customClassName.reserve(
      customClassName.size() + ns.size() + 1 + classname.size());
  customClassName.append(ns);
  customClassName.push_back('.');
  customClassName.append(classname);
  return torch::getCustomClass(customClassName);
}

void TypeParser::expect(const char* s) {
  std::string_view token = cur();
  TORCH_CHECK(
      token == s,
      "Error when parsing type ",
      pythonStr_,
      ": Expect ",
      s,
      ", but get ",
      token);
  advance();
}

// std::string_view::operator== calls memcmp to compare against the target
// string; we can do better if we specialize for a single character.
```

- **EN:** Important callable entry points in this range include expectChar, getCustomClass, expect, TORCH_CHECK, advance.
- **CN:** 这一段的重要可调用入口包括 expectChar, getCustomClass, expect, TORCH_CHECK, advance。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Type system / 类型系统, Operator schema / 算子模式, Registration / 注册机制, Declared symbols / 声明的符号, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Operator schema / 算子模式, Registration / 注册机制, Declared symbols / 声明的符号, Result propagation / 结果传递。

### Lines 281-308 / 第 281-308 行

```cpp
void TypeParser::expectChar(char c) {
  std::string_view token = cur();
  TORCH_CHECK(
      token.size() == 1 && token[0] == c,
      "Error when parsing type ",
      pythonStr_,
      ": Expect ",
      c,
      ", but get ",
      token);
  advance();
}

void TypeParser::lex() {
  // skip white spaces
  while (start_ < pythonStr_.size() && pythonStr_[start_] == ' ')
    ++start_;
  if (start_ < pythonStr_.size()) {
    if (isSpecialChar(pythonStr_[start_])) {
      next_token_ = std::string_view(pythonStr_.data() + start_++, 1);
    } else { // A word
      size_t end = start_;
      for (; end < pythonStr_.size() && !isSpecialChar(pythonStr_[end]) &&
           pythonStr_[end] != ' ';
           ++end)
        ;
      next_token_ = std::string_view(pythonStr_.data() + start_, end - start_);
      start_ = end;
```

- **EN:** Important callable entry points in this range include expectChar, TORCH_CHECK, advance, lex.
- **CN:** 这一段的重要可调用入口包括 expectChar, TORCH_CHECK, advance, lex。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 309-336 / 第 309-336 行

```cpp
    }
  }
}

std::string_view TypeParser::nextView() {
  TORCH_CHECK(
      !next_token_.empty(),
      "Empty token queue in mobile type parser.",
      "Check the format of the type string and make sure it's correct.");
  std::string_view token = cur();
  advance();
  return token;
}

std::string TypeParser::next() {
  auto token = nextView();
  return std::string(token.begin(), token.end());
}

void TypeParser::advance() {
  next_token_ = "";
  lex();
}

[[nodiscard]] std::string_view TypeParser::cur() const {
  return next_token_;
}

```

- **EN:** Important callable entry points in this range include nextView, TORCH_CHECK, advance, next, string, lex.
- **CN:** 这一段的重要可调用入口包括 nextView, TORCH_CHECK, advance, next, string, lex。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Type system / 类型系统, Operator schema / 算子模式, Mobile runtime / 移动端运行时, Declared symbols / 声明的符号, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Operator schema / 算子模式, Mobile runtime / 移动端运行时, Declared symbols / 声明的符号, Result propagation / 结果传递。

### Lines 337-348 / 第 337-348 行

```cpp
TORCH_API at::TypePtr parseType(const std::string& pythonStr) {
  at::TypeParser parser(pythonStr);
  return parser.parse();
}

TORCH_API std::vector<at::TypePtr> parseType(
    std::vector<std::string>& pythonStrs) {
  at::TypeParser parser(pythonStrs);
  return parser.parseList();
}

} // namespace c10
```

- **EN:** Important callable entry points in this range include parseType, parser.
- **CN:** 这一段的重要可调用入口包括 parseType, parser。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Type system / 类型系统, Operator schema / 算子模式, Namespace scoping / 命名空间作用域, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Operator schema / 算子模式, Namespace scoping / 命名空间作用域, Result propagation / 结果传递。

## Key Concepts / 关键概念

- **Lite interpreter runtime** — 轻量解释器运行时
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Mobile runtime** — 移动端运行时
- **Backend integration** — 后端集成
- **Registration** — 注册机制
- **Core symbols: always, type, starts, definition, isSpecialChar, strchr, TypeParser, lex** — 核心符号：always、type、starts、definition、isSpecialChar、strchr、TypeParser、lex

## Dependencies / 依赖关系

- `torch/csrc/jit/mobile/type_parser.h`
- `ATen/core/jit_type.h`
- `ATen/core/type_factory.h`
- `torch/csrc/jit/frontend/parser_constants.h`
- `torch/custom_class.h`
