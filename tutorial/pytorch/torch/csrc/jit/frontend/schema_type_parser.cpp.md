# schema_type_parser.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/frontend/schema_type_parser.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements TorchScript frontend parsing, lowering, or helper logic for schema type parser.
- 用途 (CN): 实现与 schema type parser 相关的 TorchScript 前端解析、降级或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13
```cpp
#include <torch/csrc/jit/frontend/schema_type_parser.h>

#include <ATen/core/alias_info.h>
#include <ATen/core/jit_type.h>
#include <ATen/core/symbol.h>
#include <ATen/core/type_factory.h>
#include <torch/csrc/jit/frontend/lexer.h>
#include <torch/csrc/jit/frontend/parse_string_literal.h>
#include <torch/custom_class.h>
#include <mutex>
#include <string>
#include <unordered_set>

```
- EN: Pulls in the headers needed by the schema type parser logic. Internal dependencies: `torch/csrc/jit/frontend/schema_type_parser.h`, `ATen/core/alias_info.h`, `ATen/core/jit_type.h`, `ATen/core/symbol.h`, `ATen/core/type_factory.h`, `...`; external dependencies: `mutex`, `string`, `unordered_set`.
- CN: 为 schema type parser 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/frontend/schema_type_parser.h`, `ATen/core/alias_info.h`, `ATen/core/jit_type.h`, `ATen/core/symbol.h`, `ATen/core/type_factory.h`, `...`；外部依赖：`mutex`, `string`, `unordered_set`。

### Lines 14-27
```cpp
using c10::AliasInfo;
using c10::AwaitType;
using c10::BoolType;
using c10::CapsuleType;
using c10::ComplexType;
using c10::DeviceObjType;
using c10::DictType;
using c10::FloatType;
using c10::FutureType;
using c10::GeneratorType;
using c10::IntType;
using c10::LayoutType;
using c10::ListType;
using c10::MemoryFormatType;
```
- EN: This block implements local helper logic for schema type parser. Key symbols: `c10`.
- CN: 该代码块实现与 schema type parser 相关的局部辅助逻辑。关键符号：`c10`。

### Lines 28-41
```cpp
using c10::NoneType;
using c10::NumberType;
using c10::QSchemeType;
using c10::QuantizerType;
using c10::RRefType;
using c10::ScalarTypeType;
using c10::StorageType;
using c10::StreamObjType;
using c10::StringType;
using c10::Symbol;
using c10::SymBoolType;
using c10::SymIntType;
using c10::TensorType;
using c10::TupleType;
```
- EN: This block implements local helper logic for schema type parser. Key symbols: `c10`.
- CN: 该代码块实现与 schema type parser 相关的局部辅助逻辑。关键符号：`c10`。

### Lines 42-48
```cpp
using c10::UnionType;
using c10::VarType;

namespace torch::jit {

static std::mutex& getOpaqueTypesMutex() {
  static std::mutex opaque_types_mutex;
```
- EN: This block protects shared state or ordering assumptions. Key symbols: `c10`, `getOpaqueTypesMutex`.
- CN: 该代码块保护共享状态或执行顺序假设。关键符号：`c10`, `getOpaqueTypesMutex`。

### Lines 49-56
```cpp
  return opaque_types_mutex;
}

static std::unordered_set<std::string>& getOpaqueTypes() {
  static std::unordered_set<std::string> global_opaque_types;
  return global_opaque_types;
}

```
- EN: This block produces a result or forwards a computed value; protects shared state or ordering assumptions. Key symbols: `getOpaqueTypes`.
- CN: 该代码块返回结果或转发已计算的值；保护共享状态或执行顺序假设。关键符号：`getOpaqueTypes`。

### Lines 57-66
```cpp
void registerOpaqueType(const std::string& type_name) {
  std::lock_guard<std::mutex> lock(getOpaqueTypesMutex());
  auto& global_opaque_types = getOpaqueTypes();
  auto [_, inserted] = global_opaque_types.insert(type_name);
  if (!inserted) {
    throw std::runtime_error(
        "Type '" + type_name + "' is already registered as an opaque type");
  }
}

```
- EN: This block handles conditional branches; protects shared state or ordering assumptions. Key symbols: `registerOpaqueType`, `lock`, `getOpaqueTypesMutex`, `getOpaqueTypes`, `insert`, `runtime_error`.
- CN: 该代码块处理条件分支；保护共享状态或执行顺序假设。关键符号：`registerOpaqueType`, `lock`, `getOpaqueTypesMutex`, `getOpaqueTypes`, `insert`, `runtime_error`。

### Lines 67-73
```cpp
void unregisterOpaqueType(const std::string& type_name) {
  std::lock_guard<std::mutex> lock(getOpaqueTypesMutex());
  auto& global_opaque_types = getOpaqueTypes();
  global_opaque_types.erase(type_name);
}

bool isRegisteredOpaqueType(const std::string& type_name) {
```
- EN: This block protects shared state or ordering assumptions. Key symbols: `unregisterOpaqueType`, `lock`, `getOpaqueTypesMutex`, `getOpaqueTypes`, `erase`, `isRegisteredOpaqueType`.
- CN: 该代码块保护共享状态或执行顺序假设。关键符号：`unregisterOpaqueType`, `lock`, `getOpaqueTypesMutex`, `getOpaqueTypes`, `erase`, `isRegisteredOpaqueType`。

### Lines 74-80
```cpp
  std::lock_guard<std::mutex> lock(getOpaqueTypesMutex());
  auto& global_opaque_types = getOpaqueTypes();
  return global_opaque_types.find(type_name) != global_opaque_types.end();
}

TypePtr SchemaTypeParser::parseBaseType() {
  static std::unordered_map<std::string, TypePtr> type_map = {
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `lock`, `getOpaqueTypesMutex`, `getOpaqueTypes`, `find`, `end`, `parseBaseType`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`lock`, `getOpaqueTypesMutex`, `getOpaqueTypes`, `find`, `end`, `parseBaseType`。

### Lines 81-94
```cpp
      {"Generator", c10::TypeFactory::get<GeneratorType>()},
      {"Dimname", c10::TypeFactory::get<StringType>()},
      {"ScalarType", c10::TypeFactory::get<ScalarTypeType>()},
      {"Layout", c10::TypeFactory::get<LayoutType>()},
      {"MemoryFormat", c10::TypeFactory::get<MemoryFormatType>()},
      {"Storage", c10::TypeFactory::get<StorageType>()},
      {"QScheme", c10::TypeFactory::get<QSchemeType>()},
      {"Quantizer", c10::TypeFactory::get<QuantizerType>()},
      {"ConstQuantizerPtr",
       c10::TypeFactory::get<IntType>()}, // TODO This type should be removed
                                          // from the schema parser, it should
                                          // use the custom class mechanism
                                          // instead. @jerryzh
      {"Device", c10::TypeFactory::get<DeviceObjType>()},
```
- EN: Declares core types or data containers for this file. Prominent symbols: no dominant local symbol names.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：无明显局部符号。

### Lines 95-108
```cpp
      {"DeviceIndex", c10::TypeFactory::get<IntType>()},
      {"Stream", c10::TypeFactory::get<StreamObjType>()},
      {"Scalar", c10::TypeFactory::get<NumberType>()},
      {"str", c10::TypeFactory::get<StringType>()},
      {"float", c10::TypeFactory::get<FloatType>()},
      {"complex", c10::TypeFactory::get<ComplexType>()},
      {"int", c10::TypeFactory::get<IntType>()},
      {"SymInt", c10::TypeFactory::get<SymIntType>()},
      {"bool", c10::TypeFactory::get<BoolType>()},
      {"SymBool", c10::TypeFactory::get<SymBoolType>()},
      {"None", c10::TypeFactory::get<NoneType>()},
      {"NoneType", c10::TypeFactory::get<NoneType>()},
      {"Capsule", c10::TypeFactory::get<CapsuleType>()},
      {"Any", c10::TypeFactory::get<c10::AnyType>()},
```
- EN: This block implements local helper logic for schema type parser. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 schema type parser 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 109-115
```cpp
      {"AnyClassType", c10::TypeFactory::get<c10::AnyClassType>()},
      {"AnyEnumType", c10::TypeFactory::get<c10::AnyEnumType>()},
      // PyObjectType::get() used directly because PyObjectType is excluded
      // from FORALL_DYNAMIC_TYPES (not supported on xplat/mobile)
      {"PyObject", c10::PyObjectType::get()},
  };
  auto tok = L.cur();
```
- EN: This block implements local helper logic for schema type parser. Key symbols: `get`, `cur`.
- CN: 该代码块实现与 schema type parser 相关的局部辅助逻辑。关键符号：`get`, `cur`。

### Lines 116-123
```cpp
  if (!L.nextIf(TK_NONE) && !L.nextIf(TK_NONE_TYPE)) {
    L.expect(TK_IDENT);
  }
  std::string text = tok.text();

  // Check if this might be a dotted identifier (for opaque types)
  // Keep consuming '.' + IDENT sequences to build fully qualified names
  while (L.cur().kind == '.' && L.lookahead().kind == TK_IDENT) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `nextIf`, `expect`, `text`, `cur`, `lookahead`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`nextIf`, `expect`, `text`, `cur`, `lookahead`。

### Lines 124-130
```cpp
    L.next(); // consume '.'
    auto ident_tok = L.expect(TK_IDENT);
    text += "." + ident_tok.text();
  }

  // Check if this type is registered as an opaque type first
  if (isRegisteredOpaqueType(text)) {
```
- EN: This block handles conditional branches. Key symbols: `next`, `expect`, `text`, `isRegisteredOpaqueType`.
- CN: 该代码块处理条件分支。关键符号：`next`, `expect`, `text`, `isRegisteredOpaqueType`。

### Lines 131-139
```cpp
    return c10::PyObjectType::get();
  }

  auto it = type_map.find(text);
  if (it == type_map.end()) {
    if (allow_typevars_ && !text.empty() && islower(text[0])) {
      // lower case identifiers that are not otherwise valid types
      // are treated as type variables
      return c10::TypeFactory::createNamed<VarType>(std::move(text));
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `get`, `find`, `end`, `empty`, `islower`, `move`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`get`, `find`, `end`, `empty`, `islower`, `move`。

### Lines 140-147
```cpp
    }
    if (text == "double") {
      throw(
          ErrorReport(tok.range)
          << "Use `float` instead of `double` in an operator's schema string. "
             "`float` in schema corresponds to the double type in C++");
    }
    if (text == "int64_t") {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `throw`, `ErrorReport`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`throw`, `ErrorReport`。

### Lines 148-161
```cpp
      throw(
          ErrorReport(tok.range)
          << "Use `SymInt` or `int` instead of `int64_t` in an operator's schema string. "
             "`SymInt` corresponds to c10::SymInt in C++ while `int` in schema corresponds "
             "to the int64_t type in C++.");
    }
    throw(
        ErrorReport(tok.range)
        << "unknown type specifier. Common valid schema types include "
           "Tensor, SymInt, int, float, bool, Scalar; "
           "for a full list, please see "
           "https://github.com/pytorch/pytorch/blob/main/aten/src/ATen/native/README.md#func ");
  }
  return it->second;
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `throw`, `ErrorReport`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`throw`, `ErrorReport`。

### Lines 162-170
```cpp
}

// Examples:
// Tensor(a) // Tensor is in set a
// Tensor(a!) // it is also written to
// Tensor!  // shorthand for Tensor(fresh_identifier!)
// Tensor(a! -> a|b) // Tensor is in set a, written to,
//                      and after the write is in set a AND b.
std::optional<AliasInfo> SchemaTypeParser::parseAliasAnnotation() {
```
- EN: This block advances frontend parsing or lowering state. Key symbols: `parseAliasAnnotation`.
- CN: 该代码块推进前端解析或降级状态。关键符号：`parseAliasAnnotation`。

### Lines 171-177
```cpp
  if (L.nextIf('(')) {
    // optional 'alias set annotation'
    AliasInfo alias_info;
    parseList(TK_NOTHING, '|', TK_NOTHING, [&] {
      if (L.nextIf('*')) {
        alias_info.addBeforeSet(AliasInfo::wildcardSet());

```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `nextIf`, `parseList`, `addBeforeSet`, `wildcardSet`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`nextIf`, `parseList`, `addBeforeSet`, `wildcardSet`。

### Lines 178-184
```cpp
        // If we found a wildcard, ignore all subsequent annotations
      } else if (!alias_info.isWildcardBefore()) {
        alias_info.addBeforeSet(
            Symbol::fromQualString("alias::" + L.expect(TK_IDENT).text()));
      }
    });
    if (L.nextIf('!')) {
```
- EN: This block handles conditional branches. Key symbols: `isWildcardBefore`, `addBeforeSet`, `fromQualString`, `expect`, `text`, `nextIf`.
- CN: 该代码块处理条件分支。关键符号：`isWildcardBefore`, `addBeforeSet`, `fromQualString`, `expect`, `text`, `nextIf`。

### Lines 185-192
```cpp
      alias_info.setIsWrite(true);
    }
    if (L.nextIf(TK_ARROW)) {
      // optional 'alias set annotation'
      parseList(TK_NOTHING, '|', TK_NOTHING, [&] {
        if (L.nextIf('*')) {
          alias_info.addAfterSet(AliasInfo::wildcardSet());

```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `setIsWrite`, `nextIf`, `parseList`, `addAfterSet`, `wildcardSet`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`setIsWrite`, `nextIf`, `parseList`, `addAfterSet`, `wildcardSet`。

### Lines 193-203
```cpp
          // If we found a wildcard, ignore all subsequent annotations
        } else if (!alias_info.isWildcardAfter()) {
          alias_info.addAfterSet(
              Symbol::fromQualString("alias::" + L.expect(TK_IDENT).text()));
        }
      });
    } else {
      // We didn't encounter an ->, so assume the "after set" is identical
      // to the "before set"
      AT_ASSERT(alias_info.afterSets().empty());
      for (const auto& set : alias_info.beforeSets()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `isWildcardAfter`, `addAfterSet`, `fromQualString`, `expect`, `text`, `afterSets`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`isWildcardAfter`, `addAfterSet`, `fromQualString`, `expect`, `text`, `afterSets`, `...`。

### Lines 204-214
```cpp
        alias_info.addAfterSet(set);
      }
    }
    L.expect(')');
    return alias_info;
  } else if (L.nextIf('!')) {
    AliasInfo alias_info;
    alias_info.addBeforeSet(
        Symbol::fromQualString("alias::$" + std::to_string(next_id++)));
    alias_info.setIsWrite(true);
    return alias_info;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `addAfterSet`, `expect`, `nextIf`, `addBeforeSet`, `fromQualString`, `to_string`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`addAfterSet`, `expect`, `nextIf`, `addBeforeSet`, `fromQualString`, `to_string`, `...`。

### Lines 215-223
```cpp
  } else {
    return std::nullopt;
  }
}

std::optional<at::ScalarType> SchemaTypeParser::parseTensorDType(
    const std::string& dtype) {
#define DEFINE_SCALAR_TYPE(_1, n) {#n, at::ScalarType::n},

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseTensorDType`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseTensorDType`。

### Lines 224-230
```cpp
  static std::unordered_map<std::string, at::ScalarType> type_map = {
      AT_FORALL_SCALAR_TYPES_WITH_COMPLEX_AND_QINTS(DEFINE_SCALAR_TYPE)};

#undef DEFINE_SCALAR_TYPE
  auto type = type_map.find(dtype);
  if (type != type_map.end()) {
    return type->second;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `find`, `end`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`find`, `end`。

### Lines 231-238
```cpp
  }
  return std::nullopt;
}

std::optional<c10::Device> SchemaTypeParser::tryToParseDeviceType() {
  L.expect('=');
  const std::string& dev = L.expect(TK_IDENT).text();

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `tryToParseDeviceType`, `expect`, `text`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`tryToParseDeviceType`, `expect`, `text`。

### Lines 239-245
```cpp
  if (dev == "cpu") {
    return c10::Device(at::kCPU);
  }

  if (dev == "cuda" || dev == "hpu") {
    c10::DeviceIndex device_idx = -1;
    if (L.cur().kind == ':') {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `Device`, `cur`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`Device`, `cur`。

### Lines 246-258
```cpp
      L.expect(':');
      const std::string& num = L.expect(TK_NUMBER).text();
      try {
        device_idx = static_cast<c10::DeviceIndex>(std::stoi(num));
      } catch (const std::invalid_argument&) {
        throw(
            ErrorReport(L.cur())
            << "Device index cannot be converted to integer");
      } catch (const std::out_of_range&) {
        throw(ErrorReport(L.cur()) << "Device index is too long");
      }
    }
    if (dev == "cuda") {
```
- EN: This block handles conditional branches. Key symbols: `expect`, `text`, `stoi`, `throw`, `ErrorReport`, `cur`.
- CN: 该代码块处理条件分支。关键符号：`expect`, `text`, `stoi`, `throw`, `ErrorReport`, `cur`。

### Lines 259-267
```cpp
      return c10::Device(at::kCUDA, device_idx);
    } else {
      return c10::Device(at::kHPU, device_idx);
    }
  }

  throw(ErrorReport(L.cur()) << "cannot parse device type '" << dev << "'\n");
}

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `Device`, `throw`, `ErrorReport`, `cur`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`Device`, `throw`, `ErrorReport`, `cur`。

### Lines 268-281
```cpp
std::optional<bool> SchemaTypeParser::tryToParseRequiresGrad() {
  L.expect('=');
  const std::string& num = L.expect(TK_NUMBER).text();
  try {
    return (bool)std::stoi(num);
  } catch (const std::invalid_argument&) {
    throw(
        ErrorReport(L.cur())
        << "Field requires_grad cannot be converted to integer");
  } catch (const std::out_of_range&) {
    throw(ErrorReport(L.cur()) << "Field requires_grad is too long");
  }
}

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `tryToParseRequiresGrad`, `expect`, `text`, `stoi`, `throw`, `ErrorReport`, `...`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`tryToParseRequiresGrad`, `expect`, `text`, `stoi`, `throw`, `ErrorReport`, `...`。

### Lines 282-289
```cpp
TypePtr SchemaTypeParser::parseRefinedTensor() {
  auto maybe_dtype = parseTensorDType(L.expect(TK_IDENT).text());
  AT_ASSERT(maybe_dtype);
  at::ScalarType dtype = *maybe_dtype;
  TypePtr ptr;
  L.expect('(');
  TypePtr tensor_type;
  std::optional<c10::Device> device;
```
- EN: This block advances frontend parsing or lowering state. Key symbols: `parseRefinedTensor`, `parseTensorDType`, `expect`, `text`.
- CN: 该代码块推进前端解析或降级状态。关键符号：`parseRefinedTensor`, `parseTensorDType`, `expect`, `text`。

### Lines 290-298
```cpp
  std::optional<bool> requires_grad;
  // Parse a type with either no ranks, known ranks with sizes, ranks with
  // unknown sizes, a mix of ranks with known and unknown sizes, or ranks with
  // known sizes and strides. The type might also have requires_grad and/or
  // device option. Examples of types we're handling here:
  //   Long(10, 8, 6, strides=[48, 6, 1], requires_grad=0, device=cuda:1)
  //   Float(10, *, 20, device=cuda:1)
  //   Float(requires_grad=1)
  std::vector<std::optional<int64_t>> dims;
```
- EN: This block advances frontend parsing or lowering state. Key symbols: no dominant local symbol names.
- CN: 该代码块推进前端解析或降级状态。关键符号：无明显局部符号。

### Lines 299-305
```cpp
  bool seen_strides = false;
  std::vector<int64_t> strides;
  parseList(TK_NOTHING, ',', ')', [&] {
    // Extra handling for options like 'device' and 'requires_grad'
    if (L.cur().kind == TK_IDENT && L.cur().text() != "SS") {
      const std::string& field = L.expect(TK_IDENT).text();
      if (field == "device") {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `parseList`, `cur`, `text`, `expect`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`parseList`, `cur`, `text`, `expect`。

### Lines 306-315
```cpp
        auto parsed_device = tryToParseDeviceType();
        if (parsed_device.has_value()) {
          if (device.has_value()) {
            throw(ErrorReport(L.cur()) << "'device' is specified twice");
          }
          device = parsed_device;
        }
        return;
      }
      if (field == "requires_grad") {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `tryToParseDeviceType`, `has_value`, `throw`, `ErrorReport`, `cur`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`tryToParseDeviceType`, `has_value`, `throw`, `ErrorReport`, `cur`。

### Lines 316-325
```cpp
        auto parsed_requires_grad = tryToParseRequiresGrad();
        if (parsed_requires_grad.has_value()) {
          if (requires_grad.has_value()) {
            throw(ErrorReport(L.cur()) << "'requires_grad' is specified twice");
          }
          requires_grad = parsed_requires_grad;
        }
        return;
      }
      if (field == "strides") {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `tryToParseRequiresGrad`, `has_value`, `throw`, `ErrorReport`, `cur`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`tryToParseRequiresGrad`, `has_value`, `throw`, `ErrorReport`, `cur`。

### Lines 326-339
```cpp
        seen_strides = true;
        L.expect('=');
        parseList('[', ',', ']', [&] {
          const std::string& num = L.expect(TK_NUMBER).text();
          try {
            auto stride = std::stoll(num);
            strides.push_back(stride);
          } catch (const std::invalid_argument&) {
            throw(
                ErrorReport(L.cur())
                << "The stride value cannot be converted to int");
          } catch (const std::out_of_range&) {
            throw(ErrorReport(L.cur()) << "The stride is too big");
          }
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `expect`, `parseList`, `text`, `stoll`, `push_back`, `throw`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`expect`, `parseList`, `text`, `stoll`, `push_back`, `throw`, `...`。

### Lines 340-350
```cpp
        });
        return;
      }
      throw(ErrorReport(L.cur()) << "Unexpected specifier '" << field << "'");
    }
    if (device.has_value() || requires_grad.has_value()) {
      throw(
          ErrorReport(L.cur())
          << "'device' and 'requires_grad' should come after dimensions in the type specification");
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `throw`, `ErrorReport`, `cur`, `has_value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`throw`, `ErrorReport`, `cur`, `has_value`。

### Lines 351-362
```cpp
    // Parsing ranks, supports mix of sized and unsized ranks, or, just strided
    // ranks
    if (L.cur().kind == '*') {
      dims.emplace_back(std::nullopt);
      L.next();
      if (L.cur().kind == ':') {
        throw(
            ErrorReport(L.cur()) << "Strides for unsized ranks not supported");
      }
      return;
    }
    bool shape_symbol = false;
```
- EN: This block handles conditional branches. Key symbols: `cur`, `emplace_back`, `next`, `throw`, `ErrorReport`.
- CN: 该代码块处理条件分支。关键符号：`cur`, `emplace_back`, `next`, `throw`, `ErrorReport`。

### Lines 363-376
```cpp
    if (L.cur().kind == TK_IDENT && L.cur().text_view() == "SS") {
      L.next();
      L.expect('(');
      L.expect('-');
      shape_symbol = true;
    }
    const std::string& num = L.expect(TK_NUMBER).text();
    int64_t dim = 0;
    try {
      dim = std::stoll(num);
    } catch (const std::invalid_argument&) {
      throw(ErrorReport(L.cur()) << "The number can't be converted to int");
    } catch (const std::out_of_range&) {
      throw(ErrorReport(L.cur()) << "Number is too big");
```
- EN: This block handles conditional branches. Key symbols: `cur`, `text_view`, `next`, `expect`, `text`, `stoll`, `...`.
- CN: 该代码块处理条件分支。关键符号：`cur`, `text_view`, `next`, `expect`, `text`, `stoll`, `...`。

### Lines 377-384
```cpp
    }
    if (shape_symbol) {
      L.expect(')');
      dim = -dim;
    }
    dims.emplace_back(dim);
  });
  if (seen_strides) {
```
- EN: This block handles conditional branches. Key symbols: `expect`, `emplace_back`.
- CN: 该代码块处理条件分支。关键符号：`expect`, `emplace_back`。

### Lines 385-398
```cpp
    at::IntArrayRef strides_ref(strides);
    if (strides.size() != dims.size()) {
      // note: mixing unsized ranks and ranks with strides will always trigger
      // this
      throw(
          ErrorReport(L.cur())
          << "Strides info is specified for some but not for all dimensions");
    }
    ptr = at::TensorType::create(
        dtype,
        device,
        c10::VaryingShape<int64_t>(dims),
        c10::VaryingShape<int64_t>(strides),
        requires_grad);
```
- EN: This block handles conditional branches. Key symbols: `strides_ref`, `size`, `throw`, `ErrorReport`, `cur`, `create`.
- CN: 该代码块处理条件分支。关键符号：`strides_ref`, `size`, `throw`, `ErrorReport`, `cur`, `create`。

### Lines 399-407
```cpp
  } else {
    ptr = at::TensorType::create(
        dtype,
        device,
        c10::VaryingShape<int64_t>(dims),
        c10::VaryingShape<int64_t>(dims.size()),
        requires_grad);
  }
  return ptr;
```
- EN: This block produces a result or forwards a computed value. Key symbols: `create`, `size`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`create`, `size`。

### Lines 408-414
```cpp
}

std::pair<TypePtr, std::optional<AliasInfo>> SchemaTypeParser::parseType() {
  auto r = parseFakeAndRealType();
  return std::make_pair(std::move(std::get<0>(r)), std::move(std::get<2>(r)));
}

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseType`, `parseFakeAndRealType`, `make_pair`, `move`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseType`, `parseFakeAndRealType`, `make_pair`, `move`。

### Lines 415-421
```cpp
std::tuple</*fake*/ TypePtr, /*real*/ TypePtr, std::optional<AliasInfo>>
SchemaTypeParser::parseFakeAndRealType() {
  TypePtr fake_value;
  TypePtr real_value;
  std::optional<AliasInfo> alias_info;
  // Tuple type
  if (L.cur().kind == '(') {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parseFakeAndRealType`, `cur`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parseFakeAndRealType`, `cur`。

### Lines 422-435
```cpp
    std::vector<TypePtr> types;
    parseList('(', ',', ')', [&] {
      auto r = parseType();
      types.push_back(std::move(r.first));
      if (alias_info && r.second) {
        alias_info->addContainedType(std::move(*r.second));
      }
    });
    fake_value = real_value =
        c10::TypeFactory::create<TupleType>(std::move(types));
  } else if (L.cur().kind == TK_IDENT && L.cur().text_view() == "Future") {
    L.next(); // Future
    L.expect('(');
    auto p = parseType();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parseList`, `parseType`, `push_back`, `move`, `addContainedType`, `cur`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parseList`, `parseType`, `push_back`, `move`, `addContainedType`, `cur`, `...`。

### Lines 436-444
```cpp
    auto subtype = std::move(p.first);
    auto subalias = std::move(p.second);
    L.expect(')');
    fake_value = real_value =
        c10::TypeFactory::create<FutureType>(std::move(subtype));
  } else if (L.cur().kind == TK_IDENT && L.cur().text_view() == "Await") {
    L.next(); // Await
    L.expect('(');
    auto p = parseType();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `move`, `expect`, `cur`, `text_view`, `next`, `parseType`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`move`, `expect`, `cur`, `text_view`, `next`, `parseType`。

### Lines 445-453
```cpp
    auto subtype = std::move(p.first);
    auto subalias = std::move(p.second);
    L.expect(')');
    fake_value = real_value =
        c10::TypeFactory::create<AwaitType>(std::move(subtype));
  } else if (L.cur().kind == TK_IDENT && L.cur().text_view() == "RRef") {
    L.next(); // RRef
    L.expect('(');
    auto p = parseType();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `move`, `expect`, `cur`, `text_view`, `next`, `parseType`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`move`, `expect`, `cur`, `text_view`, `next`, `parseType`。

### Lines 454-466
```cpp
    auto subtype = std::move(p.first);
    auto subalias = std::move(p.second);
    L.expect(')');
    fake_value = real_value =
        c10::TypeFactory::create<RRefType>(std::move(subtype));
  } else if (L.cur().kind == TK_IDENT && L.cur().text_view() == "Tensor") {
    L.next();
    fake_value = real_value = c10::TypeFactory::get<TensorType>();
    alias_info = parseAliasAnnotation();
  } else if (L.cur().kind == TK_IDENT && L.cur().text_view() == "Dict") {
    L.next();
    L.expect('(');
    auto key_type = parseType().first;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `move`, `expect`, `cur`, `text_view`, `next`, `parseAliasAnnotation`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`move`, `expect`, `cur`, `text_view`, `next`, `parseAliasAnnotation`, `...`。

### Lines 467-476
```cpp
    L.expect(',');
    auto value_type = parseType().first;
    L.expect(')');
    alias_info = parseAliasAnnotation();
    fake_value = real_value = c10::TypeFactory::create<DictType>(
        std::move(key_type), std::move(value_type));
  } else if (L.cur().kind == TK_IDENT && L.cur().text_view() == "Union") {
    L.next();
    L.expect('(');
    std::vector<TypePtr> types;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `expect`, `parseType`, `parseAliasAnnotation`, `move`, `cur`, `text_view`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`expect`, `parseType`, `parseAliasAnnotation`, `move`, `cur`, `text_view`, `...`。

### Lines 477-490
```cpp
    types.emplace_back(parseType().first);
    while (L.cur().kind != ')') {
      L.expect(',');
      types.emplace_back(parseType().first);
    }
    L.expect(')');
    alias_info = parseAliasAnnotation();
    fake_value = real_value =
        c10::TypeFactory::create<c10::UnionType>(std::move(types));
  } else if (
      complete_tensor_types && L.cur().kind == TK_IDENT &&
      parseTensorDType(L.cur().text())) {
    fake_value = real_value = parseRefinedTensor();
    alias_info = parseAliasAnnotation();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `emplace_back`, `parseType`, `cur`, `expect`, `parseAliasAnnotation`, `move`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`emplace_back`, `parseType`, `cur`, `expect`, `parseAliasAnnotation`, `move`, `...`。

### Lines 491-501
```cpp
  } else if (L.cur().kind == TK_IDENT && L.cur().text_view() == "__torch__") {
    L.next();
    L.expect('.');
    auto torch_tok = L.expect(TK_IDENT);
    if (torch_tok.text() != "torch") {
      throw(
          ErrorReport(torch_tok.range)
          << "Expected classes namespace but got " << torch_tok.text());
    }
    L.expect('.');
    auto classes_tok = L.expect(TK_IDENT);
```
- EN: This block handles conditional branches. Key symbols: `cur`, `text_view`, `next`, `expect`, `text`, `throw`, `...`.
- CN: 该代码块处理条件分支。关键符号：`cur`, `text_view`, `next`, `expect`, `text`, `throw`, `...`。

### Lines 502-508
```cpp
    if (classes_tok.text() != "classes") {
      throw(
          ErrorReport(classes_tok.range)
          << "Expected classes namespace but got " << classes_tok.text());
    }
    L.expect('.');
    auto ns_tok = L.expect(TK_IDENT);
```
- EN: This block handles conditional branches. Key symbols: `text`, `throw`, `ErrorReport`, `expect`.
- CN: 该代码块处理条件分支。关键符号：`text`, `throw`, `ErrorReport`, `expect`。

### Lines 509-522
```cpp
    L.expect('.');
    auto class_tok = L.expect(TK_IDENT);
    fake_value = real_value = getCustomClass(
        std::string("__torch__.torch.classes.") + ns_tok.text() + "." +
        class_tok.text());
    if (!fake_value) {
      throw(
          ErrorReport(class_tok.range) << "Unknown custom class type "
                                       << ns_tok.text() + "." + class_tok.text()
                                       << ". Please ensure it is registered.");
    }
  } else {
    real_value = parseBaseType();
    if (real_value->kind() == ScalarTypeType::Kind ||
```
- EN: Declares core types or data containers for this file. Prominent symbols: `expect`, `getCustomClass`, `string`, `text`, `throw`, `ErrorReport`, `...`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`expect`, `getCustomClass`, `string`, `text`, `throw`, `ErrorReport`, `...`。

### Lines 523-532
```cpp
        real_value->kind() == MemoryFormatType::Kind ||
        real_value->kind() == LayoutType::Kind ||
        real_value->kind() == SymIntType::Kind) {
      fake_value = c10::TypeFactory::get<IntType>();
    } else {
      fake_value = real_value;
    }
    alias_info = parseAliasAnnotation();
  }
  while (true) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `kind`, `parseAliasAnnotation`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`kind`, `parseAliasAnnotation`。

### Lines 533-539
```cpp
    if (L.cur().kind == '[' && L.lookahead().kind == ']') {
      L.next(); // [
      L.next(); // ]
      fake_value = c10::TypeFactory::create<ListType>(std::move(fake_value));
      real_value = c10::TypeFactory::create<ListType>(std::move(real_value));
      auto container = parseAliasAnnotation();
      if (alias_info) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `cur`, `lookahead`, `next`, `move`, `parseAliasAnnotation`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`cur`, `lookahead`, `next`, `move`, `parseAliasAnnotation`。

### Lines 540-553
```cpp
        if (!container) {
          container = std::optional<AliasInfo>(AliasInfo());
          container->setIsWrite(alias_info->isWrite());
        }
        container->addContainedType(std::move(*alias_info));
      }
      alias_info = std::move(container);
    } else if (L.nextIf('?')) {
      fake_value = c10::OptionalType::get(fake_value);
      real_value = c10::OptionalType::get(real_value);
    } else {
      break;
    }
  }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `AliasInfo`, `setIsWrite`, `isWrite`, `addContainedType`, `move`, `nextIf`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`AliasInfo`, `setIsWrite`, `isWrite`, `addContainedType`, `move`, `nextIf`, `...`。

### Lines 554-563
```cpp
  return std::make_tuple(
      std::move(fake_value), std::move(real_value), std::move(alias_info));
}

void SchemaTypeParser::parseList(
    int begin,
    int sep,
    int end,
    c10::function_ref<void()> callback) {
  auto r = L.cur().range;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `make_tuple`, `move`, `parseList`, `void`, `cur`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`make_tuple`, `move`, `parseList`, `void`, `cur`。

### Lines 564-571
```cpp
  if (begin != TK_NOTHING)
    L.expect(begin);
  if (L.cur().kind != end) {
    do {
      callback();
    } while (L.nextIf(sep));
  }
  if (end != TK_NOTHING)
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `expect`, `cur`, `callback`, `nextIf`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`expect`, `cur`, `callback`, `nextIf`。

### Lines 572-575
```cpp
    L.expect(end);
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Alias and mutation reasoning — the code tracks side effects and write/read interactions.
  CN: 别名与变更分析——代码会跟踪副作用以及读写关系。
- EN: Schema handling — the implementation parses or normalizes operator/function signatures.
  CN: 模式处理——实现会解析或规范化算子/函数签名。
- EN: Parsing pipeline — the logic translates source text into TorchScript syntax structures.
  CN: 解析流水线——相关逻辑将源码文本转换为 TorchScript 语法结构。
- EN: Frontend lowering — the file resolves symbols and lowers frontend constructs into IR.
  CN: 前端降级——该文件负责符号解析，并将前端结构降为 IR。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/frontend/schema_type_parser.h`, `ATen/core/alias_info.h`, `ATen/core/jit_type.h`, `ATen/core/symbol.h`, `ATen/core/type_factory.h`, `torch/csrc/jit/frontend/lexer.h`, `torch/csrc/jit/frontend/parse_string_literal.h`, `torch/custom_class.h`
- External includes / 外部头文件: `mutex`, `string`, `unordered_set`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `c10`, `getOpaqueTypesMutex`, `getOpaqueTypes`, `registerOpaqueType`, `lock`, `insert`, `runtime_error`, `unregisterOpaqueType`, `erase`, `isRegisteredOpaqueType`, `...`
