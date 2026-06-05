# parser.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/frontend/parser.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements TorchScript frontend parsing, lowering, or helper logic for parser.
- 用途 (CN): 实现与 parser 相关的 TorchScript 前端解析、降级或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```cpp
#include <torch/csrc/jit/frontend/parser.h>

#include <torch/csrc/jit/frontend/lexer.h>
#include <torch/csrc/jit/frontend/parse_string_literal.h>
#include <torch/csrc/jit/frontend/tree.h>
#include <torch/csrc/jit/frontend/tree_views.h>
#include <optional>

namespace torch::jit {
```
- EN: This block advances frontend parsing or lowering state. Key symbols: no dominant local symbol names.
- CN: 该代码块推进前端解析或降级状态。关键符号：无明显局部符号。

### Lines 10-20
```cpp

Decl mergeTypesFromTypeComment(
    const Decl& decl,
    const Decl& type_annotation_decl,
    bool is_method) {
  auto expected_num_annotations = decl.params().size();
  if (is_method) {
    // `self` argument
    expected_num_annotations -= 1;
  }
  if (expected_num_annotations != type_annotation_decl.params().size()) {
```
- EN: This block handles conditional branches. Key symbols: `mergeTypesFromTypeComment`, `params`, `size`.
- CN: 该代码块处理条件分支。关键符号：`mergeTypesFromTypeComment`, `params`, `size`。

### Lines 21-29
```cpp
    throw ErrorReport(decl.range())
        << "Number of type annotations ("
        << type_annotation_decl.params().size()
        << ") did not match the number of "
        << (is_method ? "method" : "function") << " parameters ("
        << expected_num_annotations << ')';
  }
  auto old = decl.params();
  auto _new = type_annotation_decl.params();
```
- EN: This block implements local helper logic for parser. Key symbols: `ErrorReport`, `range`, `annotations`, `params`, `size`, `parameters`.
- CN: 该代码块实现与 parser 相关的局部辅助逻辑。关键符号：`ErrorReport`, `range`, `annotations`, `params`, `size`, `parameters`。

### Lines 30-38
```cpp
  // Merge signature idents and ranges with annotation types

  std::vector<Param> new_params;
  size_t i = is_method ? 1 : 0;
  size_t j = 0;
  if (is_method) {
    new_params.push_back(old[0]);
  }
  for (; i < decl.params().size(); ++i, ++j) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `push_back`, `params`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`push_back`, `params`, `size`。

### Lines 39-47
```cpp
    new_params.emplace_back(old[i].withType(_new[j].type()));
  }
  return Decl::create(
      decl.range(),
      List<Param>::create(decl.range(), new_params),
      type_annotation_decl.return_type());
}

struct ParserImpl {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `emplace_back`, `withType`, `type`, `create`, `range`, `return_type`, `...`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`emplace_back`, `withType`, `type`, `create`, `range`, `return_type`, `...`。

### Lines 48-56
```cpp
  explicit ParserImpl(const std::shared_ptr<Source>& source)
      : L(source), shared(sharedParserData()) {}

  Ident parseIdent() {
    auto t = L.expect(TK_IDENT);
    // whenever we parse something that has a TreeView type we always
    // use its create method so that the accessors and the constructor
    // of the Compound tree are in the same place.
    return Ident::create(t.range, t.text());
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `ParserImpl`, `shared`, `sharedParserData`, `parseIdent`, `expect`, `create`, `...`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`ParserImpl`, `shared`, `sharedParserData`, `parseIdent`, `expect`, `create`, `...`。

### Lines 57-69
```cpp
  }
  TreeRef createApply(const Expr& expr) {
    TreeList attributes;
    auto range = L.cur().range;
    TreeList inputs;
    parseArguments(inputs, attributes);
    return Apply::create(
        range,
        expr,
        List<Expr>(makeList(range, std::move(inputs))),
        List<Attribute>(makeList(range, std::move(attributes))));
  }

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `createApply`, `cur`, `parseArguments`, `create`, `makeList`, `move`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`createApply`, `cur`, `parseArguments`, `create`, `makeList`, `move`。

### Lines 70-86
```cpp
  static bool followsTuple(int kind) {
    switch (kind) {
      case TK_PLUS_EQ:
      case TK_MINUS_EQ:
      case TK_TIMES_EQ:
      case TK_DIV_EQ:
      case TK_MOD_EQ:
      case TK_BIT_OR_EQ:
      case TK_BIT_AND_EQ:
      case TK_BIT_XOR_EQ:
      case TK_LSHIFT_EQ:
      case TK_RSHIFT_EQ:
      case TK_POW_EQ:
      case TK_NEWLINE:
      case '=':
      case ')':
        return true;
```
- EN: This block produces a result or forwards a computed value. Key symbols: `followsTuple`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`followsTuple`。

### Lines 87-95
```cpp
      default:
        return false;
    }
  }

  // exp | expr, | expr, expr, ...
  Expr parseExpOrExpTuple() {
    auto prefix = parseExp();
    if (L.cur().kind == ',') {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseExpOrExpTuple`, `parseExp`, `cur`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseExpOrExpTuple`, `parseExp`, `cur`。

### Lines 96-105
```cpp
      std::vector<Expr> exprs = {prefix};
      while (L.nextIf(',')) {
        if (followsTuple(L.cur().kind))
          break;
        exprs.push_back(parseExp());
      }
      auto list = List<Expr>::create(prefix.range(), exprs);
      prefix = TupleLiteral::create(list.range(), list);
    }
    return prefix;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `nextIf`, `followsTuple`, `cur`, `push_back`, `parseExp`, `create`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`nextIf`, `followsTuple`, `cur`, `push_back`, `parseExp`, `create`, `...`。

### Lines 106-119
```cpp
  }
  // things like a 1.0 or a(4) that are not unary/binary expressions
  // and have higher precedence than all of them
  TreeRef parseBaseExp() {
    TreeRef prefix;
    switch (L.cur().kind) {
      case TK_NUMBER: {
        prefix = parseConst();
      } break;
      case TK_TRUE:
      case TK_FALSE:
      case TK_NONE:
      case TK_NONE_TYPE: {
        auto k = L.cur().kind;
```
- EN: This block advances frontend parsing or lowering state. Key symbols: `parseBaseExp`, `cur`, `parseConst`.
- CN: 该代码块推进前端解析或降级状态。关键符号：`parseBaseExp`, `cur`, `parseConst`。

### Lines 120-128
```cpp
        auto r = L.cur().range;
        prefix = create_compound(k, r, {});
        L.next();
      } break;
      case '(': {
        L.next();
        if (L.nextIf(')')) {
          /// here we have the empty tuple case
          std::vector<Expr> vecExpr;
```
- EN: This block handles conditional branches. Key symbols: `cur`, `create_compound`, `next`, `nextIf`.
- CN: 该代码块处理条件分支。关键符号：`cur`, `create_compound`, `next`, `nextIf`。

### Lines 129-137
```cpp
          List<Expr> listExpr = List<Expr>::create(L.cur().range, vecExpr);
          prefix = TupleLiteral::create(L.cur().range, listExpr);
          break;
        }
        prefix = parseExpOrExpTuple();
        L.expect(')');
      } break;
      case '[': {
        auto list = parseList('[', ',', ']', &ParserImpl::parseExp);
```
- EN: This block advances frontend parsing or lowering state. Key symbols: `create`, `cur`, `parseExpOrExpTuple`, `expect`, `parseList`.
- CN: 该代码块推进前端解析或降级状态。关键符号：`create`, `cur`, `parseExpOrExpTuple`, `expect`, `parseList`。

### Lines 138-150
```cpp

        if (list.size() == 1 && (*list.begin()).kind() == TK_LIST_COMP) {
          prefix = *list.begin();
        } else {
          for (auto se : list) {
            if (se.kind() == TK_LIST_COMP) {
              throw ErrorReport(list.range())
                  << " expected a single list comprehension within '[' , ']'";
            }
          }
          prefix = ListLiteral::create(list.range(), List<Expr>(list));
        }

```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `size`, `begin`, `kind`, `ErrorReport`, `range`, `create`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`size`, `begin`, `kind`, `ErrorReport`, `range`, `create`。

### Lines 151-164
```cpp
      } break;
      case '{': {
        L.next();
        // If we have a dict literal, `keys` and `values` will store the keys
        // and values used in the object's construction. EDGE CASE: We have a
        // dict comprehension, so we'll get the first element of the dict
        // comprehension in `keys` and a list comprehension in `values`.
        // For example, `{i : chr(i + 65) for i in range(4)}` would give us
        // `i` in `keys` and `chr(i + 65) for i in range(4)` in `values`.
        // The optimal way of handling this case is to simply splice the new
        // dict comprehension together from the existing list comprehension.
        // Splicing prevents breaking changes to our API and does not require
        // the use of global variables.
        std::vector<Expr> keys;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `next`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`next`。

### Lines 165-175
```cpp
        std::vector<Expr> values;
        auto range = L.cur().range;
        if (L.cur().kind != '}') {
          do {
            keys.push_back(parseExp());
            L.expect(':');
            values.push_back(parseExp());
          } while (L.nextIf(','));
        }
        L.expect('}');
        if (keys.size() == 1 && (*values.begin()).kind() == TK_LIST_COMP) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `cur`, `push_back`, `parseExp`, `expect`, `nextIf`, `size`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`cur`, `push_back`, `parseExp`, `expect`, `nextIf`, `size`, `...`。

### Lines 176-193
```cpp
          ListComp lc(*values.begin());
          prefix = DictComp::create(
              range, *keys.begin(), lc.elt(), lc.target(), lc.iter());
        } else {
          prefix = DictLiteral::create(
              range,
              List<Expr>::create(range, keys),
              List<Expr>::create(range, values));
        }
      } break;
      case TK_STRINGLITERAL: {
        prefix = parseConcatenatedStringLiterals();
      } break;
      case TK_ELLIPSIS:
      case TK_DOTS: {
        prefix = Dots::create(L.cur().range);
        L.next();
      } break;
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `lc`, `begin`, `create`, `elt`, `target`, `iter`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`lc`, `begin`, `create`, `elt`, `target`, `iter`, `...`。

### Lines 194-211
```cpp
      default: {
        Ident name = parseIdent();
        prefix = Var::create(name.range(), name);
      } break;
    }
    while (true) {
      if (L.nextIf('.')) {
        const auto name = parseIdent();
        prefix = Select::create(name.range(), Expr(prefix), Ident(name));
      } else if (L.cur().kind == '(') {
        prefix = createApply(Expr(prefix));
      } else if (L.cur().kind == '[') {
        prefix = parseSubscript(prefix);
      } else {
        break;
      }
    }
    return prefix;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseIdent`, `create`, `range`, `nextIf`, `Expr`, `Ident`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseIdent`, `create`, `range`, `nextIf`, `Expr`, `Ident`, `...`。

### Lines 212-225
```cpp
  }
  std::optional<TreeRef> maybeParseAssignmentOp() {
    auto r = L.cur().range;
    switch (L.cur().kind) {
      case TK_PLUS_EQ:
      case TK_MINUS_EQ:
      case TK_TIMES_EQ:
      case TK_DIV_EQ:
      case TK_BIT_OR_EQ:
      case TK_BIT_AND_EQ:
      case TK_BIT_XOR_EQ:
      case TK_MOD_EQ: {
        int modifier = L.next().text()[0];
        return create_compound(modifier, r, {});
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `maybeParseAssignmentOp`, `cur`, `next`, `text`, `create_compound`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`maybeParseAssignmentOp`, `cur`, `next`, `text`, `create_compound`。

### Lines 226-237
```cpp
      } break;
      case TK_LSHIFT_EQ: {
        L.next();
        return create_compound(TK_LSHIFT, r, {});
      } break;
      case TK_RSHIFT_EQ: {
        L.next();
        return create_compound(TK_RSHIFT, r, {});
      } break;
      case TK_POW_EQ: {
        L.next();
        return create_compound(TK_POW, r, {});
```
- EN: This block produces a result or forwards a computed value. Key symbols: `next`, `create_compound`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`next`, `create_compound`。

### Lines 238-251
```cpp
      } break;
      case '=': {
        L.next();
        return create_compound('=', r, {}); // no reduction
      } break;
      default:
        return std::nullopt;
    }
  }
  TreeRef parseTrinary(
      TreeRef true_branch,
      const SourceRange& range,
      int binary_prec) {
    auto cond = parseExp();
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `next`, `create_compound`, `parseTrinary`, `parseExp`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`next`, `create_compound`, `parseTrinary`, `parseExp`。

### Lines 252-262
```cpp
    L.expect(TK_ELSE);
    auto false_branch = parseExp(binary_prec);
    return create_compound(
        TK_IF_EXPR, range, {cond, std::move(true_branch), false_branch});
  }
  // parse the longest expression whose binary operators have
  // precedence strictly greater than 'precedence'
  // precedence == 0 will parse _all_ expressions
  // this is the core loop of 'top-down precedence parsing'
  Expr parseExp() {
    return parseExp(0);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `expect`, `parseExp`, `create_compound`, `move`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`expect`, `parseExp`, `create_compound`, `move`。

### Lines 263-271
```cpp
  }
  Expr parseExp(int precedence) {
    TreeRef prefix;
    int unary_prec = 0;
    if (shared.isUnary(L.cur().kind, &unary_prec)) {
      auto kind = L.cur().kind;
      auto pos = L.cur().range;
      L.next();
      auto unary_kind = kind == '*' ? TK_STARRED
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `parseExp`, `isUnary`, `cur`, `next`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`parseExp`, `isUnary`, `cur`, `next`。

### Lines 272-286
```cpp
          : kind == '-'             ? TK_UNARY_MINUS
                                    : kind;
      auto subexp = parseExp(unary_prec);
      // fold '-' into constant numbers, so that attributes can accept
      // things like -1
      if (unary_kind == TK_UNARY_MINUS && subexp.kind() == TK_CONST) {
        prefix = Const::create(subexp.range(), "-" + Const(subexp).text());
      } else {
        prefix = create_compound(unary_kind, pos, {subexp});
      }
    } else {
      prefix = parseBaseExp();
    }
    int binary_prec = 0;
    while (shared.isBinary(L.cur().kind, &binary_prec)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; advances frontend parsing or lowering state; performs optimization-oriented rewriting. Key symbols: `parseExp`, `kind`, `create`, `range`, `Const`, `text`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；推进前端解析或降级状态；执行面向优化的改写。关键符号：`parseExp`, `kind`, `create`, `range`, `Const`, `text`, `...`。

### Lines 287-296
```cpp
      if (binary_prec <= precedence) // not allowed to parse something which is
        // not greater than 'precedence'
        break;

      int kind = L.cur().kind;
      auto pos = L.cur().range;
      L.next();
      if (shared.isRightAssociative(kind))
        binary_prec--;

```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `cur`, `next`, `isRightAssociative`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`cur`, `next`, `isRightAssociative`。

### Lines 297-306
```cpp
      if (kind == TK_NOTIN) {
        // NB: `not in` is just `not( in )`, so we don't introduce new tree view
        // but just make it a nested call in our tree view structure
        prefix = create_compound(TK_IN, pos, {prefix, parseExp(binary_prec)});
        prefix = create_compound(TK_NOT, pos, {prefix});
        continue;
      }

      // special case for trinary operator
      if (kind == TK_IF) {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `create_compound`, `parseExp`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`create_compound`, `parseExp`。

### Lines 307-316
```cpp
        prefix = parseTrinary(prefix, pos, binary_prec);
        continue;
      }

      if (kind == TK_FOR) {
        // TK_FOR targets should only parse exprs prec greater than 4, which
        // only includes subset of Exprs that suppose to be on the LHS according
        // to the python grammar
        // https://docs.python.org/3/reference/grammar.html
        auto target = parseLHSExp();
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `parseTrinary`, `parseLHSExp`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`parseTrinary`, `parseLHSExp`。

### Lines 317-325
```cpp
        L.expect(TK_IN);
        auto iter = parseExp();
        prefix = ListComp::create(pos, Expr(prefix), target, iter);
        continue;
      }

      prefix = create_compound(kind, pos, {prefix, parseExp(binary_prec)});
    }
    return Expr(prefix);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `expect`, `parseExp`, `create`, `Expr`, `create_compound`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`expect`, `parseExp`, `create`, `Expr`, `create_compound`。

### Lines 326-336
```cpp
  }

  void parseSequence(
      int begin,
      int sep,
      int end,
      const std::function<void()>& parse) {
    if (begin != TK_NOTHING) {
      L.expect(begin);
    }
    while (end != L.cur().kind) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; advances frontend parsing or lowering state. Key symbols: `parseSequence`, `void`, `expect`, `cur`.
- CN: 该代码块处理条件分支；遍历集合或图结构；推进前端解析或降级状态。关键符号：`parseSequence`, `void`, `expect`, `cur`。

### Lines 337-347
```cpp
      parse();
      if (!L.nextIf(sep)) {
        if (end != TK_NOTHING) {
          L.expect(end);
        }
        return;
      }
    }
    L.expect(end);
  }
  template <typename T>
```
- EN: Builds a reusable template/helper layer around parser. Symbols: `parse`, `nextIf`, `expect`.
- CN: 围绕 parser 构建可复用的模板或辅助层。符号：`parse`, `nextIf`, `expect`。

### Lines 348-357
```cpp
  List<T> parseList(int begin, int sep, int end, T (ParserImpl::*parse)()) {
    auto r = L.cur().range;
    std::vector<T> elements;
    parseSequence(
        begin, sep, end, [&] { elements.emplace_back((this->*parse)()); });
    return List<T>::create(r, elements);
  }

  Const parseConst() {
    auto range = L.cur().range;
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseList`, `cur`, `parseSequence`, `emplace_back`, `create`, `parseConst`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseList`, `cur`, `parseSequence`, `emplace_back`, `create`, `parseConst`。

### Lines 358-366
```cpp
    auto t = L.expect(TK_NUMBER);
    return Const::create(t.range, t.text());
  }

  StringLiteral parseConcatenatedStringLiterals() {
    auto range = L.cur().range;
    std::string ss;
    while (L.cur().kind == TK_STRINGLITERAL) {
      auto literal_range = L.cur().range;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `expect`, `create`, `text`, `parseConcatenatedStringLiterals`, `cur`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`expect`, `create`, `text`, `parseConcatenatedStringLiterals`, `cur`。

### Lines 367-375
```cpp
      ss.append(parseStringLiteral(literal_range, L.next().text()));
    }
    return StringLiteral::create(range, ss);
  }

  Expr parseAttributeValue() {
    return parseExp();
  }

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `append`, `parseStringLiteral`, `next`, `text`, `create`, `parseAttributeValue`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`append`, `parseStringLiteral`, `next`, `text`, `create`, `parseAttributeValue`, `...`。

### Lines 376-388
```cpp
  void parseArguments(TreeList& inputs, TreeList& attributes) {
    parseSequence('(', ',', ')', [&] {
      if (L.cur().kind == TK_IDENT && L.lookahead().kind == '=') {
        auto ident = parseIdent();
        L.expect('=');
        auto v = parseAttributeValue();
        attributes.push_back(Attribute::create(ident.range(), Ident(ident), v));
      } else {
        inputs.push_back(parseExp());
      }
    });
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parseArguments`, `parseSequence`, `cur`, `lookahead`, `parseIdent`, `expect`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parseArguments`, `parseSequence`, `cur`, `lookahead`, `parseIdent`, `expect`, `...`。

### Lines 389-399
```cpp
  // parse LHS acceptable exprs, which only includes subset of Exprs that prec
  // is greater than 4 according to the python grammar
  Expr parseLHSExp() {
    return parseExp(4);
  }

  // Parse expr's of the form [a:], [:b], [a:b], [:] and all variations with
  // "::"
  Expr parseSubscriptExp() {
    TreeRef first, second, third;
    auto range = L.cur().range;
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseLHSExp`, `parseExp`, `parseSubscriptExp`, `cur`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseLHSExp`, `parseExp`, `parseSubscriptExp`, `cur`。

### Lines 400-408
```cpp
    if (L.cur().kind != ':') {
      first = parseExp();
    }
    if (L.nextIf(':')) {
      if (L.cur().kind != ',' && L.cur().kind != ']' && L.cur().kind != ':') {
        second = parseExp();
      }
      if (L.nextIf(':')) {
        if (L.cur().kind != ',' && L.cur().kind != ']') {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `cur`, `parseExp`, `nextIf`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`cur`, `parseExp`, `nextIf`。

### Lines 409-418
```cpp
          third = parseExp();
        }
      }
      auto maybe_first = first ? Maybe<Expr>::create(range, Expr(first))
                               : Maybe<Expr>::create(range);
      auto maybe_second = second ? Maybe<Expr>::create(range, Expr(second))
                                 : Maybe<Expr>::create(range);
      auto maybe_third = third ? Maybe<Expr>::create(range, Expr(third))
                               : Maybe<Expr>::create(range);
      return SliceExpr::create(range, maybe_first, maybe_second, maybe_third);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseExp`, `create`, `Expr`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseExp`, `create`, `Expr`。

### Lines 419-427
```cpp
    } else {
      return Expr(first);
    }
  }

  TreeRef parseSubscript(const TreeRef& value) {
    const auto range = L.cur().range;

    auto subscript_exprs =
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `Expr`, `parseSubscript`, `cur`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`Expr`, `parseSubscript`, `cur`。

### Lines 428-436
```cpp
        parseList('[', ',', ']', &ParserImpl::parseSubscriptExp);

    const auto whole_range =
        SourceRange(range.source(), range.start(), L.cur().range.start());
    return Subscript::create(whole_range, Expr(value), subscript_exprs);
  }

  Maybe<Expr> maybeParseTypeAnnotation() {
    if (L.nextIf(':')) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parseList`, `SourceRange`, `source`, `start`, `cur`, `create`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parseList`, `SourceRange`, `source`, `start`, `cur`, `create`, `...`。

### Lines 437-446
```cpp
      // NB: parseExp must not be called inline, since argument evaluation order
      // changes when L.cur().range is mutated with respect to the parseExp()
      // call.
      auto expr = parseExp();
      return Maybe<Expr>::create(expr.range(), expr);
    } else {
      return Maybe<Expr>::create(L.cur().range);
    }
  }

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state; performs optimization-oriented rewriting. Key symbols: `parseExp`, `create`, `range`, `cur`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态；执行面向优化的改写。关键符号：`parseExp`, `create`, `range`, `cur`。

### Lines 447-455
```cpp
  TreeRef parseFormalParam(bool kwarg_only) {
    auto ident = parseIdent();
    TreeRef type = maybeParseTypeAnnotation();
    TreeRef def;
    if (L.nextIf('=')) {
      // NB: parseExp must not be called inline, since argument evaluation order
      // changes when L.cur().range is mutated with respect to the parseExp()
      // call.
      auto expr = parseExp();
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state; performs optimization-oriented rewriting. Key symbols: `parseFormalParam`, `parseIdent`, `maybeParseTypeAnnotation`, `nextIf`, `parseExp`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态；执行面向优化的改写。关键符号：`parseFormalParam`, `parseIdent`, `maybeParseTypeAnnotation`, `nextIf`, `parseExp`。

### Lines 456-467
```cpp
      def = Maybe<Expr>::create(expr.range(), expr);
    } else {
      def = Maybe<Expr>::create(L.cur().range);
    }
    return Param::create(
        type->range(),
        Ident(ident),
        Maybe<Expr>(type),
        Maybe<Expr>(def),
        kwarg_only);
  }

```
- EN: This block produces a result or forwards a computed value. Key symbols: `create`, `range`, `cur`, `Ident`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`create`, `range`, `cur`, `Ident`。

### Lines 468-477
```cpp
  Param parseBareTypeAnnotation() {
    auto type = parseExp();
    return Param::create(
        type.range(),
        Ident::create(type.range(), ""),
        Maybe<Expr>::create(type.range(), type),
        Maybe<Expr>::create(type.range()),
        /*kwarg_only=*/false);
  }

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseBareTypeAnnotation`, `parseExp`, `create`, `range`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseBareTypeAnnotation`, `parseExp`, `create`, `range`。

### Lines 478-490
```cpp
  Decl parseTypeComment() {
    auto range = L.cur().range;
    L.expect(TK_TYPE_COMMENT);
    auto param_types =
        parseList('(', ',', ')', &ParserImpl::parseBareTypeAnnotation);
    TreeRef return_type;
    if (L.nextIf(TK_ARROW)) {
      auto return_type_range = L.cur().range;
      return_type = Maybe<Expr>::create(return_type_range, parseExp());
    } else {
      return_type = Maybe<Expr>::create(L.cur().range);
    }
    return Decl::create(range, param_types, Maybe<Expr>(return_type));
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseTypeComment`, `cur`, `expect`, `parseList`, `nextIf`, `create`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseTypeComment`, `cur`, `expect`, `parseList`, `nextIf`, `create`, `...`。

### Lines 491-499
```cpp
  }

  // 'first' has already been parsed since expressions can exist
  // alone on a line:
  // first[,other,lhs] = rhs
  TreeRef parseAssign(const Expr& lhs) {
    auto type = maybeParseTypeAnnotation();
    auto maybeOp = maybeParseAssignmentOp();
    if (maybeOp) {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `parseAssign`, `maybeParseTypeAnnotation`, `maybeParseAssignmentOp`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`parseAssign`, `maybeParseTypeAnnotation`, `maybeParseAssignmentOp`。

### Lines 500-509
```cpp
      // There is an assignment operator, parse the RHS and generate the
      // assignment.
      auto rhs = parseExpOrExpTuple();
      if (maybeOp.value()->kind() == '=') {
        std::vector<Expr> lhs_list = {lhs};
        while (L.nextIf('=')) {
          lhs_list.push_back(rhs);
          rhs = parseExpOrExpTuple();
        }
        if (type.present() && lhs_list.size() > 1) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parseExpOrExpTuple`, `value`, `kind`, `nextIf`, `push_back`, `present`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parseExpOrExpTuple`, `value`, `kind`, `nextIf`, `push_back`, `present`, `...`。

### Lines 510-522
```cpp
          throw ErrorReport(type.range())
              << "Annotated multiple assignment is not supported in python";
        }
        L.expect(TK_NEWLINE);
        return Assign::create(
            lhs.range(),
            List<Expr>::create(lhs_list[0].range(), lhs_list),
            Maybe<Expr>::create(rhs.range(), rhs),
            type);
      } else {
        L.expect(TK_NEWLINE);
        // this is an augmented assignment
        if (lhs.kind() == TK_TUPLE_LITERAL) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `ErrorReport`, `range`, `expect`, `create`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`ErrorReport`, `range`, `expect`, `create`, `kind`。

### Lines 523-533
```cpp
          throw ErrorReport(lhs.range())
              << " augmented assignment can only have one LHS expression";
        }
        return AugAssign::create(
            lhs.range(), lhs, AugAssignKind(*maybeOp), Expr(rhs));
      }
    } else {
      // There is no assignment operator, so this is of the form `lhs : <type>`
      TORCH_INTERNAL_ASSERT(type.present());
      L.expect(TK_NEWLINE);
      return Assign::create(
```
- EN: This block produces a result or forwards a computed value. Key symbols: `ErrorReport`, `range`, `create`, `AugAssignKind`, `Expr`, `present`, `...`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`ErrorReport`, `range`, `create`, `AugAssignKind`, `Expr`, `present`, `...`。

### Lines 534-544
```cpp
          lhs.range(),
          List<Expr>::create(lhs.range(), {lhs}),
          Maybe<Expr>::create(lhs.range()),
          type);
    }
  }

  TreeRef parseStmt(bool in_class = false) {
    switch (L.cur().kind) {
      case TK_IF:
        return parseIf();
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `range`, `create`, `parseStmt`, `cur`, `parseIf`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`range`, `create`, `parseStmt`, `cur`, `parseIf`。

### Lines 545-554
```cpp
      case TK_WHILE:
        return parseWhile();
      case TK_FOR:
        return parseFor();
      case TK_GLOBAL: {
        auto range = L.next().range;
        auto idents =
            parseList(TK_NOTHING, ',', TK_NOTHING, &ParserImpl::parseIdent);
        L.expect(TK_NEWLINE);
        return Global::create(range, idents);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseWhile`, `parseFor`, `next`, `parseList`, `expect`, `create`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseWhile`, `parseFor`, `next`, `parseList`, `expect`, `create`。

### Lines 555-565
```cpp
      }
      case TK_RETURN: {
        auto range = L.next().range;
        Expr value = L.cur().kind != TK_NEWLINE
            ? parseExpOrExpTuple()
            : Expr(create_compound(TK_NONE, range, {}));
        L.expect(TK_NEWLINE);
        return Return::create(range, value);
      }
      case TK_RAISE: {
        auto range = L.next().range;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `next`, `cur`, `parseExpOrExpTuple`, `Expr`, `create_compound`, `expect`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`next`, `cur`, `parseExpOrExpTuple`, `Expr`, `create_compound`, `expect`, `...`。

### Lines 566-574
```cpp
        auto expr = parseExp();
        L.expect(TK_NEWLINE);
        return Raise::create(range, expr);
      }
      case TK_ASSERT: {
        auto range = L.next().range;
        auto cond = parseExp();
        Maybe<Expr> maybe_first = Maybe<Expr>::create(range);
        if (L.nextIf(',')) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseExp`, `expect`, `create`, `next`, `nextIf`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseExp`, `expect`, `create`, `next`, `nextIf`。

### Lines 575-584
```cpp
          auto msg = parseExp();
          maybe_first = Maybe<Expr>::create(range, Expr(msg));
        }
        L.expect(TK_NEWLINE);
        return Assert::create(range, cond, maybe_first);
      }
      case TK_BREAK: {
        auto range = L.next().range;
        L.expect(TK_NEWLINE);
        return Break::create(range);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseExp`, `create`, `Expr`, `expect`, `next`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseExp`, `create`, `Expr`, `expect`, `next`。

### Lines 585-594
```cpp
      }
      case TK_CONTINUE: {
        auto range = L.next().range;
        L.expect(TK_NEWLINE);
        return Continue::create(range);
      }
      case TK_PASS: {
        auto range = L.next().range;
        L.expect(TK_NEWLINE);
        return Pass::create(range);
```
- EN: This block produces a result or forwards a computed value. Key symbols: `next`, `expect`, `create`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`next`, `expect`, `create`。

### Lines 595-604
```cpp
      }
      case TK_DEF: {
        return parseFunction(/*is_method=*/in_class);
      }
      case TK_DELETE: {
        auto range = L.next().range;
        auto targets =
            parseList(TK_NOTHING, ',', TK_NOTHING, &ParserImpl::parseExp);
        L.expect(TK_NEWLINE);
        return Delete::create(range, targets);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseFunction`, `next`, `parseList`, `expect`, `create`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseFunction`, `next`, `parseList`, `expect`, `create`。

### Lines 605-615
```cpp
      }
      case TK_WITH: {
        return parseWith();
      }
      default: {
        auto lhs = parseExpOrExpTuple();
        if (L.cur().kind != TK_NEWLINE) {
          return parseAssign(lhs);
        } else {
          L.expect(TK_NEWLINE);
          return ExprStmt::create(lhs.range(), lhs);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseWith`, `parseExpOrExpTuple`, `cur`, `parseAssign`, `expect`, `create`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseWith`, `parseExpOrExpTuple`, `cur`, `parseAssign`, `expect`, `create`, `...`。

### Lines 616-624
```cpp
        }
      }
    }
  }

  WithItem parseWithItem() {
    auto target = parseExp();

    if (L.cur().kind == TK_AS) {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `parseWithItem`, `parseExp`, `cur`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`parseWithItem`, `parseExp`, `cur`。

### Lines 625-634
```cpp
      // If the current token is TK_AS, this with item is of the form
      // "expression as target".
      auto token = L.expect(TK_AS);
      Ident ident = parseIdent();
      auto var = Var::create(ident.range(), ident);
      return WithItem::create(
          token.range, target, Maybe<Var>::create(ident.range(), var));
    } else {
      // If not, this with item is of the form "expression".
      return WithItem::create(
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `expect`, `parseIdent`, `create`, `range`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`expect`, `parseIdent`, `create`, `range`。

### Lines 635-643
```cpp
          target.range(), target, Maybe<Var>::create(target.range()));
    }
  }

  TreeRef parseIf(bool expect_if = true) {
    auto r = L.cur().range;
    if (expect_if)
      L.expect(TK_IF);
    auto cond = parseExp();
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `range`, `create`, `parseIf`, `cur`, `expect`, `parseExp`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`range`, `create`, `parseIf`, `cur`, `expect`, `parseExp`。

### Lines 644-654
```cpp
    L.expect(':');
    auto true_branch = parseStatements(/*expect_indent=*/true);
    auto false_branch = makeList(L.cur().range, {});
    if (L.nextIf(TK_ELSE)) {
      L.expect(':');
      false_branch = parseStatements(/*expect_indent=*/true);
    } else if (L.nextIf(TK_ELIF)) {
      // NB: this needs to be a separate statement, since the call to parseIf
      // mutates the lexer state, and thus causes a heap-use-after-free in
      // compilers which evaluate argument expressions LTR
      auto range = L.cur().range;
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `expect`, `parseStatements`, `makeList`, `cur`, `nextIf`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`expect`, `parseStatements`, `makeList`, `cur`, `nextIf`。

### Lines 655-663
```cpp
      false_branch = makeList(range, {parseIf(false)});
    }
    return If::create(
        r, Expr(cond), List<Stmt>(true_branch), List<Stmt>(false_branch));
  }
  TreeRef parseWhile() {
    auto r = L.cur().range;
    L.expect(TK_WHILE);
    auto cond = parseExp();
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `makeList`, `parseIf`, `create`, `Expr`, `parseWhile`, `cur`, `...`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`makeList`, `parseIf`, `create`, `Expr`, `parseWhile`, `cur`, `...`。

### Lines 664-672
```cpp
    L.expect(':');
    auto body = parseStatements(/*expect_indent=*/true);
    return While::create(r, Expr(cond), List<Stmt>(body));
  }

  TreeRef parseFor() {
    auto r = L.cur().range;
    L.expect(TK_FOR);
    auto targets = parseList(TK_NOTHING, ',', TK_IN, &ParserImpl::parseLHSExp);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `expect`, `parseStatements`, `create`, `Expr`, `parseFor`, `cur`, `...`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`expect`, `parseStatements`, `create`, `Expr`, `parseFor`, `cur`, `...`。

### Lines 673-682
```cpp
    auto itrs = parseList(TK_NOTHING, ',', ':', &ParserImpl::parseExp);
    auto body = parseStatements(/*expect_indent=*/true);
    return For::create(r, targets, itrs, body);
  }

  TreeRef parseWith() {
    auto r = L.cur().range;
    // Parse "with expression [as target][, expression [as target]]*:".
    L.expect(TK_WITH);
    auto targets = parseList(TK_NOTHING, ',', ':', &ParserImpl::parseWithItem);
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseList`, `parseStatements`, `create`, `parseWith`, `cur`, `expect`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseList`, `parseStatements`, `create`, `parseWith`, `cur`, `expect`。

### Lines 683-697
```cpp
    // Parse the body.
    auto body = parseStatements(/*expect_indent=*/true);
    return With::create(r, targets, body);
  }

  TreeRef parseStatements(bool expect_indent, bool in_class = false) {
    auto r = L.cur().range;
    if (expect_indent) {
      L.expect(TK_INDENT);
    }
    TreeList stmts;
    do {
      stmts.push_back(parseStmt(in_class));
    } while (!L.nextIf(TK_DEDENT));
    return create_compound(TK_LIST, r, std::move(stmts));
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseStatements`, `create`, `cur`, `expect`, `push_back`, `parseStmt`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseStatements`, `create`, `cur`, `expect`, `push_back`, `parseStmt`, `...`。

### Lines 698-706
```cpp
  }

  Maybe<Expr> parseReturnAnnotation() {
    if (L.nextIf(TK_ARROW)) {
      // Exactly one expression for return type annotation
      auto return_type_range = L.cur().range;
      return Maybe<Expr>::create(return_type_range, parseExp());
    } else {
      return Maybe<Expr>::create(L.cur().range);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseReturnAnnotation`, `nextIf`, `cur`, `create`, `parseExp`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseReturnAnnotation`, `nextIf`, `cur`, `create`, `parseExp`。

### Lines 707-715
```cpp
    }
  }

  List<Param> parseFormalParams() {
    auto r = L.cur().range;
    std::vector<Param> params;
    bool kwarg_only = false;
    parseSequence('(', ',', ')', [&] {
      if (!kwarg_only && L.nextIf('*')) {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `parseFormalParams`, `cur`, `parseSequence`, `nextIf`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`parseFormalParams`, `cur`, `parseSequence`, `nextIf`。

### Lines 716-729
```cpp
        kwarg_only = true;
      } else {
        params.emplace_back(parseFormalParam(kwarg_only));
      }
    });
    return List<Param>::create(r, params);
  }
  Decl parseDecl() {
    // Parse return type annotation
    List<Param> paramlist = parseFormalParams();
    TreeRef return_type;
    Maybe<Expr> return_annotation = parseReturnAnnotation();
    L.expect(':');
    return Decl::create(
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `emplace_back`, `parseFormalParam`, `create`, `parseDecl`, `parseFormalParams`, `parseReturnAnnotation`, `...`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`emplace_back`, `parseFormalParam`, `create`, `parseDecl`, `parseFormalParams`, `parseReturnAnnotation`, `...`。

### Lines 730-739
```cpp
        paramlist.range(), List<Param>(paramlist), return_annotation);
  }

  TreeRef parseClass() {
    L.expect(TK_CLASS_DEF);
    const auto name = parseIdent();
    Maybe<Expr> superclass = Maybe<Expr>::create(name.range());
    if (L.nextIf('(')) {
      // Only support inheriting from NamedTuple right now.
      auto id = parseExp();
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `range`, `parseClass`, `expect`, `parseIdent`, `create`, `nextIf`, `...`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`range`, `parseClass`, `expect`, `parseIdent`, `create`, `nextIf`, `...`。

### Lines 740-749
```cpp
      superclass = Maybe<Expr>::create(id.range(), id);
      L.expect(')');
    }
    L.expect(':');
    const auto statements =
        parseStatements(/*expect_indent=*/true, /*in_class=*/true);
    return ClassDef::create(
        name.range(), name, superclass, List<Stmt>(statements));
  }

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `create`, `range`, `expect`, `parseStatements`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`create`, `range`, `expect`, `parseStatements`。

### Lines 750-759
```cpp
  TreeRef parseFunction(bool is_method) {
    L.expect(TK_DEF);
    auto name = parseIdent();
    auto decl = parseDecl();

    TreeRef stmts_list;
    if (L.nextIf(TK_INDENT)) {
      // Handle type annotations specified in a type comment as the first line
      // of the function.
      if (L.cur().kind == TK_TYPE_COMMENT) {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `parseFunction`, `expect`, `parseIdent`, `parseDecl`, `nextIf`, `cur`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`parseFunction`, `expect`, `parseIdent`, `parseDecl`, `nextIf`, `cur`。

### Lines 760-769
```cpp
        auto type_annotation_decl = Decl(parseTypeComment());
        L.expect(TK_NEWLINE);
        decl = mergeTypesFromTypeComment(decl, type_annotation_decl, is_method);
      }

      stmts_list = parseStatements(false);
    } else {
      // Special case: the Python grammar allows one-line functions with a
      // single statement.
      if (L.cur().kind == TK_TYPE_COMMENT) {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `Decl`, `parseTypeComment`, `expect`, `mergeTypesFromTypeComment`, `parseStatements`, `cur`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`Decl`, `parseTypeComment`, `expect`, `mergeTypesFromTypeComment`, `parseStatements`, `cur`。

### Lines 770-778
```cpp
        auto type_annotation_decl = Decl(parseTypeComment());
        decl = mergeTypesFromTypeComment(decl, type_annotation_decl, is_method);
      }

      TreeList stmts;
      stmts.push_back(parseStmt(is_method));
      stmts_list = create_compound(TK_LIST, L.cur().range, std::move(stmts));
    }

```
- EN: This block advances frontend parsing or lowering state. Key symbols: `Decl`, `parseTypeComment`, `mergeTypesFromTypeComment`, `push_back`, `parseStmt`, `create_compound`, `...`.
- CN: 该代码块推进前端解析或降级状态。关键符号：`Decl`, `parseTypeComment`, `mergeTypesFromTypeComment`, `push_back`, `parseStmt`, `create_compound`, `...`。

### Lines 779-792
```cpp
    return Def::create(
        name.range(), Ident(name), Decl(decl), List<Stmt>(stmts_list));
  }
  Lexer& lexer() {
    return L;
  }

 private:
  // short helpers to create nodes
  TreeRef create_compound(
      int kind,
      const SourceRange& range,
      TreeList&& trees) {
    return Compound::create(kind, range, std::move(trees));
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `create`, `range`, `Ident`, `Decl`, `lexer`, `create_compound`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`create`, `range`, `Ident`, `Decl`, `lexer`, `create_compound`, `...`。

### Lines 793-803
```cpp
  }
  TreeRef makeList(const SourceRange& range, TreeList&& trees) {
    return create_compound(TK_LIST, range, std::move(trees));
  }
  Lexer L;
  SharedParserData& shared;
};

Parser::Parser(const std::shared_ptr<Source>& src)
    : pImpl(new ParserImpl(src)) {}

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `makeList`, `create_compound`, `move`, `Parser`, `pImpl`, `ParserImpl`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`makeList`, `create_compound`, `move`, `Parser`, `pImpl`, `ParserImpl`。

### Lines 804-813
```cpp
Parser::~Parser() = default;

TreeRef Parser::parseFunction(bool is_method) {
  return pImpl->parseFunction(is_method);
}
TreeRef Parser::parseClass() {
  return pImpl->parseClass();
}
Lexer& Parser::lexer() {
  return pImpl->lexer();
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `~Parser`, `parseFunction`, `parseClass`, `lexer`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`~Parser`, `parseFunction`, `parseClass`, `lexer`。

### Lines 814-822
```cpp
}
Decl Parser::parseTypeComment() {
  return pImpl->parseTypeComment();
}
Expr Parser::parseExp() {
  return pImpl->parseExp();
}

} // namespace torch::jit
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseTypeComment`, `parseExp`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseTypeComment`, `parseExp`。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Schema handling — the implementation parses or normalizes operator/function signatures.
  CN: 模式处理——实现会解析或规范化算子/函数签名。
- EN: Parsing pipeline — the logic translates source text into TorchScript syntax structures.
  CN: 解析流水线——相关逻辑将源码文本转换为 TorchScript 语法结构。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Inlining — the implementation merges callees or blocks into their call sites.
  CN: 内联——实现会把被调用者或代码块并入调用位置。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/frontend/parser.h`, `torch/csrc/jit/frontend/lexer.h`, `torch/csrc/jit/frontend/parse_string_literal.h`, `torch/csrc/jit/frontend/tree.h`, `torch/csrc/jit/frontend/tree_views.h`
- External includes / 外部头文件: `optional`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `mergeTypesFromTypeComment`, `params`, `size`, `ErrorReport`, `range`, `annotations`, `parameters`, `push_back`, `emplace_back`, `withType`, `...`
