# symbolic_shape_cache.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/symbolic_shape_cache.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for symbolic shape cache, including graph analysis and rewrites.
- 用途 (CN): 实现与 symbolic shape cache 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8
```cpp
#include <torch/csrc/jit/passes/symbolic_shape_analysis.h>
#include <torch/csrc/jit/passes/symbolic_shape_cache.h>
#include <torch/csrc/lazy/core/cache.h>

#include <utility>

// SHAPE CACHING CODE

```
- EN: This block implements local helper logic for symbolic shape cache. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 symbolic shape cache 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 9-15
```cpp
namespace torch::jit {
namespace {
using CanonicalArg = std::variant<CanonicalizedSymbolicShape, IValue>;
using CanonicalArgVec = std::vector<CanonicalArg>;
using CanonicalRet = std::vector<CanonicalizedSymbolicShape>;
using ShapeCacheKey = std::tuple<c10::OperatorName, CanonicalArgVec>;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `CanonicalArg`, `CanonicalArgVec`, `CanonicalRet`, `ShapeCacheKey`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`CanonicalArg`, `CanonicalArgVec`, `CanonicalRet`, `ShapeCacheKey`。

### Lines 16-22
```cpp
CanonicalArgVec cannonicalizeVec(
    const std::vector<SSAInput>& arg_vec,
    std::unordered_map<int64_t, int64_t>& ss_map,
    bool deep_copy = true) {
  CanonicalArgVec canonical_args;
  canonical_args.reserve(arg_vec.size());
  for (auto& arg : arg_vec) {
```
- EN: This block iterates over collections or graph structures. Key symbols: `cannonicalizeVec`, `reserve`, `size`.
- CN: 该代码块遍历集合或图结构。关键符号：`cannonicalizeVec`, `reserve`, `size`。

### Lines 23-34
```cpp
    if (const IValue* iv = std::get_if<IValue>(&arg)) {
      if (deep_copy) {
        canonical_args.emplace_back(iv->deepcopy());
      } else {
        canonical_args.emplace_back(*iv);
      }
    } else {
      auto& ss = std::get<at::SymbolicShape>(arg);
      canonical_args.emplace_back(CanonicalizedSymbolicShape(ss, ss_map));
    }
  }
  return canonical_args;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `emplace_back`, `deepcopy`, `CanonicalizedSymbolicShape`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`emplace_back`, `deepcopy`, `CanonicalizedSymbolicShape`。

### Lines 35-42
```cpp
}

std::vector<CanonicalizedSymbolicShape> cannonicalizeVec(
    const std::vector<at::SymbolicShape>& ret_vec,
    std::unordered_map<int64_t, int64_t>& ss_map) {
  std::vector<CanonicalizedSymbolicShape> canonical_rets;
  canonical_rets.reserve(ret_vec.size());
  for (auto& ss : ret_vec) {
```
- EN: This block iterates over collections or graph structures. Key symbols: `cannonicalizeVec`, `reserve`, `size`.
- CN: 该代码块遍历集合或图结构。关键符号：`cannonicalizeVec`, `reserve`, `size`。

### Lines 43-54
```cpp
    canonical_rets.emplace_back(ss, ss_map);
  }
  return canonical_rets;
}

struct ArgumentsHasher {
  size_t operator()(const ShapeCacheKey& cacheKey) const {
    // TODO: ignore arguments that are not used in shape function (not needed
    // initially)
    auto& op_name = std::get<0>(cacheKey);
    auto& arg_vec = std::get<1>(cacheKey);

```
- EN: Declares core types or data containers for this file. Prominent symbols: `emplace_back`, `ArgumentsHasher`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`emplace_back`, `ArgumentsHasher`。

### Lines 55-63
```cpp
    size_t hash_val = c10::hash<c10::OperatorName>()(op_name);

    hash_val = at::hash_combine(std::hash<size_t>{}(arg_vec.size()), hash_val);
    for (const CanonicalArg& arg : arg_vec) {
      size_t cur_arg = 0;
      if (const IValue* ival = std::get_if<IValue>(&arg)) {
        // IValue doesn't hash List (as Python doesn't), so we will do a custom
        // list hash
        if (ival->isList()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `hash_combine`, `size`, `isList`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`hash_combine`, `size`, `isList`。

### Lines 64-77
```cpp
          TORCH_INTERNAL_ASSERT(ival->isIntList(), "Unexpected Args in List");
          cur_arg = ival->toListRef().size();
          for (const IValue& elem_ival : ival->toListRef()) {
            cur_arg = at::hash_combine(cur_arg, IValue::hash(elem_ival));
          }
        } else {
          cur_arg = IValue::hash(ival);
        }
      } else {
        cur_arg = std::get<CanonicalizedSymbolicShape>(arg).hash();
      }
      hash_val = at::hash_combine(hash_val, cur_arg);
    }
    return hash_val;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `isIntList`, `toListRef`, `size`, `hash_combine`, `hash`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`isIntList`, `toListRef`, `size`, `hash_combine`, `hash`。

### Lines 78-85
```cpp
  }
};

using ShapeCache = lazy::Cache<
    ShapeCacheKey,
    std::vector<CanonicalizedSymbolicShape>,
    ArgumentsHasher>;

```
- EN: This block implements local helper logic for symbolic shape cache. Key symbols: `ShapeCache`.
- CN: 该代码块实现与 symbolic shape cache 相关的局部辅助逻辑。关键符号：`ShapeCache`。

### Lines 86-92
```cpp
constexpr size_t kShapeCacheSize = 1024;
ShapeCache shapeCache(kShapeCacheSize);

ShapeCacheKey get_cache_key(
    const FunctionSchema* schema,
    const std::vector<SSAInput>& arg_vec,
    std::unordered_map<int64_t, int64_t>& ss_map,
```
- EN: This block advances frontend parsing or lowering state. Key symbols: `shapeCache`, `get_cache_key`.
- CN: 该代码块推进前端解析或降级状态。关键符号：`shapeCache`, `get_cache_key`。

### Lines 93-99
```cpp
    bool deep_copy = true) {
  CanonicalArgVec canonical_args = cannonicalizeVec(arg_vec, ss_map, deep_copy);
  return std::make_tuple(schema->operator_name(), std::move(canonical_args));
}

} // namespace

```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `cannonicalizeVec`, `make_tuple`, `move`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`cannonicalizeVec`, `make_tuple`, `move`。

### Lines 100-106
```cpp
TORCH_API void cache_shape_function(
    const FunctionSchema* schema,
    const std::vector<SSAInput>& arg_vec,
    const std::vector<at::SymbolicShape>& ret_vec) {
  // TODO: compare perf using std::vector<std::tuple<int64_t, int64_t>>
  auto ss_map = std::unordered_map<int64_t, int64_t>();
  auto cache_key = get_cache_key(schema, arg_vec, ss_map, /* deep_copy */ true);
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover symbolic shape cache behavior. Symbols: `cache_shape_function`, `get_cache_key`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 symbolic shape cache 的行为。符号：`cache_shape_function`, `get_cache_key`。

### Lines 107-118
```cpp
  auto can_ret_vec = std::make_shared<std::vector<CanonicalizedSymbolicShape>>(
      cannonicalizeVec(ret_vec, ss_map));
  shapeCache.Add(std::move(cache_key), std::move(can_ret_vec));
}

TORCH_API std::optional<std::vector<at::SymbolicShape>>
get_cached_shape_function(
    const FunctionSchema* schema,
    const std::vector<SSAInput>& arg_vec) {
  // TODO: compare perf using std::vector<std::tuple<int64_t, int64_t>> for both
  // ss_map and inverse_ss_map
  auto ss_map = std::unordered_map<int64_t, int64_t>();
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover symbolic shape cache behavior. Symbols: `cannonicalizeVec`, `Add`, `move`, `get_cached_shape_function`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 symbolic shape cache 的行为。符号：`cannonicalizeVec`, `Add`, `move`, `get_cached_shape_function`。

### Lines 119-126
```cpp
  auto cache_key =
      get_cache_key(schema, arg_vec, ss_map, /* deep_copy */ false);
  auto cached_ret_vec = shapeCache.Get(cache_key);
  if (cached_ret_vec == nullptr) {
    return std::nullopt;
  }
  // Decanonicalize the return values
  auto inverse_ss_map = std::unordered_map<int64_t, int64_t>();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `get_cache_key`, `Get`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`get_cache_key`, `Get`。

### Lines 127-134
```cpp
  for (auto& ss_val : ss_map) {
    inverse_ss_map[ss_val.second] = ss_val.first;
  }
  std::vector<at::SymbolicShape> ret_vec;
  for (auto& css : *cached_ret_vec) {
    ret_vec.emplace_back(css.toSymbolicShape(inverse_ss_map));
  }
  return ret_vec;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `emplace_back`, `toSymbolicShape`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`emplace_back`, `toSymbolicShape`。

### Lines 135-141
```cpp
}

// Function only to access the cache, used for testing
TORCH_API void clear_shape_cache() {
  shapeCache.Clear();
}

```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover symbolic shape cache behavior. Symbols: `clear_shape_cache`, `Clear`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 symbolic shape cache 的行为。符号：`clear_shape_cache`, `Clear`。

### Lines 142-148
```cpp
TORCH_API size_t get_shape_cache_size() {
  return shapeCache.Numel();
}

void CanonicalizedSymbolicShape::init(
    const c10::SymbolicShape& orig_shape,
    std::unordered_map<int64_t, int64_t>& ss_map) {
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover symbolic shape cache behavior. Symbols: `get_shape_cache_size`, `Numel`, `init`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 symbolic shape cache 的行为。符号：`get_shape_cache_size`, `Numel`, `init`。

### Lines 149-156
```cpp
  auto sizes = orig_shape.sizes();
  if (!sizes) {
    values_ = std::nullopt;
    return;
  }
  values_ = std::vector<int64_t>();
  int64_t cur_symbolic_index = -static_cast<int64_t>(ss_map.size()) - 1;
  for (auto& cur_shape : *sizes) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `sizes`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`sizes`, `size`。

### Lines 157-163
```cpp
    if (cur_shape.is_static()) {
      values_->push_back(cur_shape.static_size());
    } else {
      // Check for aliasing
      auto it = ss_map.find(cur_shape.value());

      if (it == ss_map.end()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `is_static`, `push_back`, `static_size`, `find`, `value`, `end`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`is_static`, `push_back`, `static_size`, `find`, `value`, `end`。

### Lines 164-173
```cpp
        values_->push_back(cur_symbolic_index);
        ss_map.insert({cur_shape.value(), cur_symbolic_index});
        cur_symbolic_index--;
      } else {
        values_->push_back(it->second);
      }
    }
  }
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `push_back`, `insert`, `value`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`push_back`, `insert`, `value`。

### Lines 174-180
```cpp
c10::SymbolicShape CanonicalizedSymbolicShape::toSymbolicShape(
    std::unordered_map<int64_t, int64_t>& inverse_ss_map) const {
  if (!values_.has_value()) {
    return c10::SymbolicShape();
  }
  std::vector<at::ShapeSymbol> sizes;
  for (long long cur_val : *values_) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `toSymbolicShape`, `has_value`, `SymbolicShape`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`toSymbolicShape`, `has_value`, `SymbolicShape`。

### Lines 181-189
```cpp
    if (cur_val >= 0) {
      sizes.push_back(at::ShapeSymbol::fromStaticSize(cur_val));
      continue;
    }
    auto res = inverse_ss_map.find(cur_val);
    if (res != inverse_ss_map.end()) {
      sizes.push_back(at::ShapeSymbol::fromStaticSize(res->second));
    } else {
      auto new_symbol = at::ShapeSymbol::newSymbol();
```
- EN: This block handles conditional branches. Key symbols: `push_back`, `fromStaticSize`, `find`, `end`, `newSymbol`.
- CN: 该代码块处理条件分支。关键符号：`push_back`, `fromStaticSize`, `find`, `end`, `newSymbol`。

### Lines 190-196
```cpp
      inverse_ss_map.insert({cur_val, new_symbol.value()});
      sizes.push_back(new_symbol);
    }
  }
  return c10::SymbolicShape(std::move(sizes));
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `insert`, `value`, `push_back`, `SymbolicShape`, `move`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`insert`, `value`, `push_back`, `SymbolicShape`, `move`。

### Lines 197-203
```cpp
size_t CanonicalizedSymbolicShape::hash() const {
  if (!values_.has_value()) {
    return 0x8cc80c80; // random value to prevent hash collisions
  }
  return c10::hash<std::vector<int64_t>>()(values_.value());
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `hash`, `has_value`, `value`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`hash`, `has_value`, `value`。

### Lines 204-209
```cpp
bool operator==(
    const CanonicalizedSymbolicShape& a,
    const CanonicalizedSymbolicShape& b) {
  return a.values_ == b.values_;
}
} // namespace torch::jit
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Alias and mutation reasoning — the code tracks side effects and write/read interactions.
  CN: 别名与变更分析——代码会跟踪副作用以及读写关系。
- EN: Schema handling — the implementation parses or normalizes operator/function signatures.
  CN: 模式处理——实现会解析或规范化算子/函数签名。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Pass orchestration — the file plugs logic into the TorchScript optimization pipeline.
  CN: Pass 编排——该文件把相关逻辑接入 TorchScript 优化流水线。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/symbolic_shape_analysis.h`, `torch/csrc/jit/passes/symbolic_shape_cache.h`, `torch/csrc/lazy/core/cache.h`
- External includes / 外部头文件: `utility`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `CanonicalArg`, `CanonicalArgVec`, `CanonicalRet`, `ShapeCacheKey`, `cannonicalizeVec`, `reserve`, `size`, `emplace_back`, `deepcopy`, `CanonicalizedSymbolicShape`, `...`
