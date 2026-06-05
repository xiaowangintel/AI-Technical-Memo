# node_hashing.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/ir/node_hashing.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or manipulates the TorchScript/JIT intermediate representation, including graphs, nodes, blocks, and analysis utilities. This specific file centers on `node_hashing.cpp`.
- **Purpose (CN)**: 定义或操作 TorchScript/JIT 中间表示，包括图、节点、块及其分析工具。 该文件具体围绕 `node_hashing.cpp` 展开。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <torch/csrc/jit/ir/ir.h>

#include <algorithm>

#include <ATen/core/symbol.h>
#include <c10/util/Exception.h>
#include <c10/util/hash.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/ir/node_hashing.h>

namespace torch::jit {

namespace {

bool tensorEqual(const at::Tensor& lhs, const at::Tensor& rhs) {
  // type_equal doesn't distinguish between mkldnn/pytorch cpu tensors,
  // and we dont want to coalesce mkldnn tensors bc they do layout
  // transformations based on usage
  if (lhs.is_mkldnn() || rhs.is_mkldnn()) {
    return false;
  }
  if (lhs.is_nested() || rhs.is_nested()) {
    return false;
  }
  // If device is not equal, lhs.equal(rhs) would throw an error.
  if (lhs.device() != rhs.device()) {
    return false;
  }
```

- **EN:** It enters or references namespace scopes such as torch::jit, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include tensorEqual.
- **CN:** 这一段的重要可调用入口包括 tensorEqual。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 29-56 / 第 29-56 行

```cpp
  return lhs.options().type_equal(rhs.options()) && lhs.equal(rhs);
}

bool typeListEqual(
    const std::vector<TypePtr>& lhs,
    const std::vector<TypePtr>& rhs) {
  if (lhs.size() != rhs.size())
    return false;
  for (const auto i : c10::irange(lhs.size())) {
    if (*lhs[i] != *rhs[i]) {
      return false;
    }
  }
  return true;
}

template <typename attribute_type> // int64_t, bool, double
bool attributesEqual(attribute_type a1, attribute_type a2) {
  return a1 == a2;
}

bool attributesEqual(const at::Tensor& a1, const at::Tensor& a2) {
  return tensorEqual(a1, a2);
}

bool ivaluesEqual(const IValue& a1, const IValue& a2);

bool attributesEqual(
```

- **EN:** Important callable entry points in this range include typeListEqual, attributesEqual, tensorEqual, ivaluesEqual.
- **CN:** 这一段的重要可调用入口包括 typeListEqual, attributesEqual, tensorEqual, ivaluesEqual。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 57-84 / 第 57-84 行

```cpp
    const std::vector<at::Tensor>& lhs,
    const std::vector<at::Tensor>& rhs) {
  if (lhs.size() != rhs.size())
    return false;
  return std::equal(lhs.begin(), lhs.end(), rhs.begin(), tensorEqual);
}

bool attributesEqual(at::ArrayRef<IValue> a1, at::ArrayRef<IValue> a2) {
  if (a1.size() != a2.size()) {
    return false;
  }
  for (const auto i : c10::irange(a1.size())) {
    if (!ivaluesEqual(a1[i], a2[i])) {
      return false;
    }
  }
  return true;
}

bool attributesEqual(const IValue& a1, const IValue& a2) {
  return ivaluesEqual(a1, a2);
}

// this is not a general-purpose comparison of IValues, it only covers the
// ivalues that are allowed as attributes, and it does not check type
// equivalence of containers.
bool ivaluesEqual(const IValue& a1, const IValue& a2) {
  if (a1.tagKind() != a2.tagKind()) {
```

- **EN:** Important callable entry points in this range include equal, attributesEqual, ivaluesEqual.
- **CN:** 这一段的重要可调用入口包括 equal, attributesEqual, ivaluesEqual。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 85-112 / 第 85-112 行

```cpp
    return false;
  }
  if (a1.isInt()) {
    return a1.toInt() == a2.toInt();
  }
  if (a1.isBool()) {
    return a1.toBool() == a2.toBool();
  }
  if (a1.isDouble()) {
    return a1.toDouble() == a2.toDouble();
  }
  if (a1.isTensor()) {
    return attributesEqual(a1.toTensor(), a2.toTensor());
  }
  if (a1.isNone()) {
    return true;
  }
  if (a1.isString()) {
    return a1.toStringRef() == a2.toStringRef();
  }
  if (a1.isList()) {
    return attributesEqual(a1.toListRef(), a2.toListRef());
  }
  if (a1.isTuple()) {
    at::ArrayRef<IValue> a1_elem = a1.toTupleRef().elements();
    at::ArrayRef<IValue> a2_elem = a2.toTupleRef().elements();
    return attributesEqual(a1_elem, a2_elem);
  }
```

- **EN:** Important callable entry points in this range include attributesEqual.
- **CN:** 这一段的重要可调用入口包括 attributesEqual。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 113-140 / 第 113-140 行

```cpp
  if (a1.isGenericDict()) {
    auto a1_dict = a1.toGenericDict();
    auto a2_dict = a2.toGenericDict();
    if (a1_dict.size() != a2_dict.size()) {
      return false;
    }

    auto it_a1 = a1_dict.begin();
    auto it_a2 = a2_dict.begin();

    while (it_a1 != a1_dict.end()) {
      const auto& e_a1 = *it_a1;
      const auto& e_a2 = *it_a2;

      if (!ivaluesEqual(e_a1.key(), e_a2.key()) ||
          !ivaluesEqual(e_a1.value(), e_a2.value())) {
        return false;
      }
      it_a1++;
      it_a2++;
    }
    return true;
  }
  if (a1.isEnum()) {
    return a1.toEnumHolder() == a2.toEnumHolder();
  }
  if (a1.isObject()) {
    return &a1.toObjectRef() == &a2.toObjectRef();
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 141-168 / 第 141-168 行

```cpp
  }
  if (a1.isGenerator()) {
    return a1.toGenerator() == a2.toGenerator();
  }
  TORCH_INTERNAL_ASSERT(false);
}

// Check whether two nodes have the same attributes in CSE.
// This function may be too conservative for general use.
// Do NOT support g/gs attributes.
bool attributesEqualCSE(const Node* lhs, const Node* rhs) {
  AT_ASSERT(lhs != nullptr);
  AT_ASSERT(rhs != nullptr);
  // One has attributes, the other does not.
  if (lhs->hasAttributes() != rhs->hasAttributes())
    return false;
  // Neither has attributes.
  if (!lhs->hasAttributes() && !rhs->hasAttributes())
    return true;

  auto lnames = lhs->attributeNames();
  auto rnames = rhs->attributeNames();
  std::sort(lnames.begin(), lnames.end());
  std::sort(rnames.begin(), rnames.end());
  if (lnames != rnames)
    return false;

  for (auto name : lnames) {
```

- **EN:** Important callable entry points in this range include TORCH_INTERNAL_ASSERT, attributesEqualCSE, AT_ASSERT, sort.
- **CN:** 这一段的重要可调用入口包括 TORCH_INTERNAL_ASSERT, attributesEqualCSE, AT_ASSERT, sort。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 169-196 / 第 169-196 行

```cpp
    if (lhs->kindOf(name) != rhs->kindOf(name))
      return false;

#define COMPARE_ATTRIBUTEVALUE(selector)                            \
  case AttributeKind::selector: {                                   \
    if (!attributesEqual(lhs->selector(name), rhs->selector(name))) \
      return false;                                                 \
  } break;

    switch (lhs->kindOf(name)) {
      COMPARE_ATTRIBUTEVALUE(f)
      COMPARE_ATTRIBUTEVALUE(c)
      COMPARE_ATTRIBUTEVALUE(fs)
      COMPARE_ATTRIBUTEVALUE(cs)
      COMPARE_ATTRIBUTEVALUE(i)
      COMPARE_ATTRIBUTEVALUE(is)
      COMPARE_ATTRIBUTEVALUE(s)
      COMPARE_ATTRIBUTEVALUE(ss)
      COMPARE_ATTRIBUTEVALUE(t)
      COMPARE_ATTRIBUTEVALUE(ts)
      COMPARE_ATTRIBUTEVALUE(ival)
      case AttributeKind::ty:
        if (*lhs->ty(name) != *rhs->ty(name)) {
          return false;
        }
        break;
      case AttributeKind::tys:
        if (!typeListEqual(lhs->tys(name), rhs->tys(name))) {
```

- **EN:** Important callable entry points in this range include COMPARE_ATTRIBUTEVALUE.
- **CN:** 这一段的重要可调用入口包括 COMPARE_ATTRIBUTEVALUE。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 197-224 / 第 197-224 行

```cpp
          return false;
        }
        break;
      case AttributeKind::g:
      case AttributeKind::gs:
        return false;
    }

#undef COMPARE_ATTRIBUTEVALUE
  }

  return true;
}

} // anonymous namespace

// Makes a hash that hashes the input Value, the output type
// as well as the node attributes
size_t HashNode::operator()(const Node* k) const {
  AT_ASSERT(k != nullptr);
  size_t constant_hash = 0;
  if (k->kind() == prim::Constant) {
    TypePtr type = k->output()->type();
    if (type->isSubtypeOf(*NumberType::get()) &&
        k->kindOf(attr::value) == AttributeKind::i) {
      constant_hash = std::hash<int64_t>{}(k->i(attr::value));
    } else if (
        type->isSubtypeOf(*NumberType::get()) &&
```

- **EN:** Important callable entry points in this range include AT_ASSERT.
- **CN:** 这一段的重要可调用入口包括 AT_ASSERT。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 225-252 / 第 225-252 行

```cpp
        k->kindOf(attr::value) == AttributeKind::f) {
      constant_hash = std::hash<double>{}(k->f(attr::value));
    } else if (
        type->isSubtypeOf(*NumberType::get()) &&
        k->kindOf(attr::value) == AttributeKind::c) {
      constant_hash = c10::hash<c10::complex<double>>{}(k->c(attr::value));
    } else if (type->isSubtypeOf(*BoolType::get())) {
      constant_hash = std::hash<bool>{}(k->i(attr::value));
    }
  }
  return get_hash(
      k->kind(),
      fmap(k->outputs(), [](const Value* v) { return v->type()->kind(); }),
      fmap(k->inputs(), [](const Value* v) { return v->unique(); }),
      constant_hash);
}

// Checks that two nodes have the same inputs, output types
// and node attributes.
bool EqualNode::operator()(const Node* lhs, const Node* rhs) const {
  if (lhs == nullptr && rhs == nullptr)
    return true;
  if (lhs == nullptr || rhs == nullptr)
    return false;

  if (lhs->kind() != rhs->kind())
    return false;

```

- **EN:** Important callable entry points in this range include get_hash, fmap.
- **CN:** 这一段的重要可调用入口包括 get_hash, fmap。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 253-280 / 第 253-280 行

```cpp
  // Check whether the output types are the same.
  auto lhs_outputs = lhs->outputs();
  auto rhs_outputs = rhs->outputs();
  if (lhs_outputs.size() != rhs_outputs.size())
    return false;
  for (const auto i : c10::irange(lhs_outputs.size())) {
    const auto& lt = lhs_outputs[i]->type();
    const auto& rt = rhs_outputs[i]->type();
    if (!(lt == rt || *lt == *rt))
      return false;
  }

  // Check whether the inputs are the same.
  auto lhs_inputs = lhs->inputs();
  auto rhs_inputs = rhs->inputs();
  if (lhs_inputs.size() != rhs_inputs.size())
    return false;
  if (!std::equal(lhs_inputs.begin(), lhs_inputs.end(), rhs_inputs.begin()))
    return false;

  if (!attributesEqualCSE(lhs, rhs))
    return false;

  // Check if the blocks contained in a op are the same
  if (lhs->blocks().size() != rhs->blocks().size()) {
    return false;
  }
  for (size_t i = 0; i < lhs->blocks().size(); ++i) {
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 281-289 / 第 281-289 行

```cpp
    if (lhs->blocks()[i] != rhs->blocks()[i]) {
      return false;
    }
  }

  return true;
}

} // namespace torch::jit
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Control-flow blocks / 控制流块, Operator schema / 算子模式, Namespace scoping / 命名空间作用域, Result propagation / 结果传递, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Control-flow blocks / 控制流块, Operator schema / 算子模式, Namespace scoping / 命名空间作用域, Result propagation / 结果传递, Branching logic / 分支逻辑。

## Key Concepts / 关键概念

- **TorchScript IR internals** — TorchScript IR 内部结构
- **Nodes and values** — 节点与值
- **Control-flow blocks** — 控制流块
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Registration** — 注册机制
- **Core symbols: tensorEqual, typeListEqual, attributesEqual, ivaluesEqual, equal, TORCH_INTERNAL_ASSERT, attributesEqualCSE, AT_ASSERT** — 核心符号：tensorEqual、typeListEqual、attributesEqual、ivaluesEqual、equal、TORCH_INTERNAL_ASSERT、attributesEqualCSE、AT_ASSERT

## Dependencies / 依赖关系

- `torch/csrc/jit/ir/ir.h`
- `ATen/core/symbol.h`
- `c10/util/Exception.h`
- `c10/util/hash.h`
- `c10/util/irange.h`
- `torch/csrc/jit/ir/node_hashing.h`
