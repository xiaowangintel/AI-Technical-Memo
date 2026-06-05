# lexer.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/frontend/lexer.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements TorchScript frontend parsing, lowering, or helper logic for lexer.
- 用途 (CN): 实现与 lexer 相关的 TorchScript 前端解析、降级或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6
```cpp
#include <torch/csrc/jit/frontend/lexer.h>

#include <cstring>
#include <string>
#include <unordered_map>

```
- EN: Pulls in the headers needed by the lexer logic. Internal dependencies: `torch/csrc/jit/frontend/lexer.h`; external dependencies: `cstring`, `string`, `unordered_map`.
- CN: 为 lexer 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/frontend/lexer.h`；外部依赖：`cstring`, `string`, `unordered_map`。

### Lines 7-16
```cpp
namespace torch::jit {

static const std::unordered_map<int, int> binary_prec = {
    {TK_IF, 1},
    {TK_FOR, 1},
    {TK_AND, 2},
    {TK_OR, 2},
    // reserve a level for unary not
    {TK_IN, 4},
    {TK_NOTIN, 4},
```
- EN: This block implements local helper logic for lexer. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 lexer 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 17-26
```cpp
    {'<', 4},
    {'>', 4},
    {TK_IS, 4},
    {TK_ISNOT, 4},
    {TK_EQ, 4},
    {TK_LE, 4},
    {TK_GE, 4},
    {TK_NE, 4},
    {'|', 5},
    {'^', 6},
```
- EN: This block implements local helper logic for lexer. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 lexer 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 27-36
```cpp
    {'&', 7},
    {TK_LSHIFT, 8},
    {TK_RSHIFT, 8},
    {'+', 9},
    {'-', 9},
    {'*', 10},
    {'/', 10},
    {TK_FLOOR_DIV, 10},
    {'%', 10},
    {'@', 10},
```
- EN: This block implements local helper logic for lexer. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 lexer 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 37-45
```cpp
    {TK_POW, 11},
};

static const std::unordered_map<int, int> unary_prec = {
    {TK_NOT, 3},
    {'~', 3},
    {'-', 10},
    {'*', 10},
};
```
- EN: This block implements local helper logic for lexer. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 lexer 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 46-51
```cpp

bool SharedParserData::isUnary(int kind, int* prec) {
  auto it = unary_prec.find(kind);
  if (it != unary_prec.end()) {
    *prec = it->second;
    return true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `isUnary`, `find`, `end`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`isUnary`, `find`, `end`。

### Lines 52-56
```cpp
  }
  return false;
}
bool SharedParserData::isBinary(int kind, int* prec) {
  auto it = binary_prec.find(kind);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `isBinary`, `find`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`isBinary`, `find`。

### Lines 57-61
```cpp
  if (it != binary_prec.end()) {
    *prec = it->second;
    return true;
  }
  return false;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `end`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`end`。

### Lines 62-66
```cpp
}

C10_EXPORT int stringToKind(const std::string& str) {
  static std::unordered_map<std::string, int> str_to_kind = []() {
    std::unordered_map<std::string, int> ret_str_to_kind;
```
- EN: This block implements local helper logic for lexer. Key symbols: `stringToKind`.
- CN: 该代码块实现与 lexer 相关的局部辅助逻辑。关键符号：`stringToKind`。

### Lines 67-72
```cpp
    ret_str_to_kind.reserve(std::strlen(valid_single_char_tokens));
    for (const char* tok = valid_single_char_tokens; *tok; tok++) {
      ret_str_to_kind[std::string(1, *tok)] = static_cast<unsigned char>(*tok);
    }
#define DEFINE_CASE(tok, _, str) \
  if (std::string(str) != "")    \
```
- EN: This block handles conditional branches; iterates over collections or graph structures; advances frontend parsing or lowering state. Key symbols: `reserve`, `strlen`, `string`.
- CN: 该代码块处理条件分支；遍历集合或图结构；推进前端解析或降级状态。关键符号：`reserve`, `strlen`, `string`。

### Lines 73-79
```cpp
    ret_str_to_kind[str] = tok;
    TC_FORALL_TOKEN_KINDS(DEFINE_CASE)
#undef DEFINE_CASE
    return ret_str_to_kind;
  }();
  try {
    return str_to_kind.at(str);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：无明显局部符号。

### Lines 80-84
```cpp
  } catch (std::out_of_range&) {
    throw std::out_of_range("unknown token in stringToKind");
  }
}

```
- EN: This block advances frontend parsing or lowering state. Key symbols: `out_of_range`.
- CN: 该代码块推进前端解析或降级状态。关键符号：`out_of_range`。

### Lines 85-91
```cpp
C10_EXPORT std::string kindToString(int kind) {
  if (kind < 256)
    return std::string(1, static_cast<char>(kind));
  switch (kind) {
#define DEFINE_CASE(tok, str, _) \
  case tok:                      \
    return str;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `kindToString`, `string`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`kindToString`, `string`。

### Lines 92-98
```cpp
    TC_FORALL_TOKEN_KINDS(DEFINE_CASE)
#undef DEFINE_CASE
    default:
      TORCH_CHECK(false, "Unknown kind: ", kind);
  }
}

```
- EN: This block advances frontend parsing or lowering state. Key symbols: no dominant local symbol names.
- CN: 该代码块推进前端解析或降级状态。关键符号：无明显局部符号。

### Lines 99-103
```cpp
C10_EXPORT SharedParserData& sharedParserData() {
  static SharedParserData data; // safely handles multi-threaded init
  return data;
}

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `sharedParserData`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`sharedParserData`。

### Lines 104-104
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: Parsing pipeline — the logic translates source text into TorchScript syntax structures.
  CN: 解析流水线——相关逻辑将源码文本转换为 TorchScript 语法结构。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/frontend/lexer.h`
- External includes / 外部头文件: `cstring`, `string`, `unordered_map`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `isUnary`, `find`, `end`, `isBinary`, `stringToKind`, `reserve`, `strlen`, `string`, `out_of_range`, `kindToString`, `...`
