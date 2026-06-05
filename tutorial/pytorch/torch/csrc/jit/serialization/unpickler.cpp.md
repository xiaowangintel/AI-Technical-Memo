# unpickler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/serialization/unpickler.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Serializes IValues, module state, or bytecode into portable binary/container representations.
- **Purpose (CN)**: 把 IValue、模块状态或字节码序列化为可移植的二进制/容器表示。
## Line-by-Line Analysis / 逐行分析

### Lines 1-36
```cpp
#include <ATen/ATen.h>
#include <ATen/core/Dict.h>
#ifdef USE_RPC
#include <torch/csrc/distributed/rpc/rref_context.h>
#endif
#include <torch/csrc/jit/api/function_impl.h>
#include <torch/csrc/jit/mobile/type_parser.h>
#include <torch/csrc/jit/serialization/storage_context.h>
#include <torch/csrc/jit/serialization/unpickler.h>
#include <torch/csrc/utils/byte_order.h>
#include <string>
#include <utility>

namespace torch::jit {

using ::c10::IValue;

static void restoreAccurateTypeTagsIfPossible(const IValue& root) {
  if (root.isObject()) {
    restoreAccurateTypeTags(root, root.type());
  }
}

// Pickled objects are stored in a form compatible with Python pickling.
// In torchscript List[T]/Dict[K, V] are statically typed and contain
// dynamic type tags that allow T, K, and V to be recovered. But this
// info is not stored in the Python pickling information. However, we
// can recover this information from the static type of the top-level
// object being unpickled, because we have a record of the type of the
// objects it contains as attributes.
// `IfPossible` - we can only do this recovery when we have an object as
// the top-level unpickled thing (which is guaranteed for Modules, but
// not for torch.load/torch.save). Otherwise we do not know the types
// of the contained objects and cannot restore the tags.
void restoreAccurateTypeTags(const IValue& root, const TypePtr& type_tag) {
  struct Work {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/distributed/rpc/rref_context.h, torch/csrc/jit/api/function_impl.h, torch/csrc/jit/mobile/type_parser.h, and 3 more; ATen/c10 facilities such as ATen/ATen.h, ATen/core/Dict.h; standard-library headers such as string, utility. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem. It introduces or extends Work, which define the primary data structures or interfaces for this portion of the file. This chunk defines `restoreAccurateTypeTags`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/distributed/rpc/rref_context.h、torch/csrc/jit/api/function_impl.h、torch/csrc/jit/mobile/type_parser.h 等共 6 项；ATen/c10 基础设施，如 ATen/ATen.h、ATen/core/Dict.h；标准库头文件，如 string、utility。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。 它引入或扩展了 Work，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `restoreAccurateTypeTags`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 37-72
```cpp
    TypePtr type;
    IValue value;
  };
  std::vector<Work> to_process = {{type_tag, root}};
  std::unordered_set<const void*> scanned;
  while (!to_process.empty()) {
    Work w = std::move(to_process.back());
    to_process.pop_back();
    // ensure we only scan each pointer value once, otherwise this
    // can become exponential (and if we allow recursive data in the future,
    // it would not terminate).
    if (w.value.isPtrType()) {
      const void* key = w.value.internalToPointer();
      auto it = scanned.find(key);
      if (it != scanned.end()) {
        continue;
      }
      scanned.emplace_hint(it, key);
    }
    auto kind = w.type->kind();
    if (auto dyn = w.type->castRaw<c10::DynamicType>()) {
      kind = dyn->dynamicKind();
    }
    switch (kind) {
      case TensorType::Kind:
      case StorageType::Kind:
      case NumberType::Kind:
      case FloatType::Kind:
      case ComplexType::Kind:
      case IntType::Kind:
      case NoneType::Kind:
      case GeneratorType::Kind:
      case QuantizerType::Kind:
      case BoolType::Kind:
      case VarType::Kind:
      case CapsuleType::Kind:
```
- **EN**: This chunk continues `restoreAccurateTypeTags` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case.
- **CN**: 这一段延续了 `restoreAccurateTypeTags`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。

### Lines 73-108
```cpp
      case PyObjectType::Kind:
      case StringType::Kind:
      case FunctionType::Kind:
      case DeviceObjType::Kind:
      case StreamObjType::Kind:
      case QSchemeType::Kind:
      case LayoutType::Kind:
      case MemoryFormatType::Kind:
      case ScalarTypeType::Kind:
      case RRefType::Kind:
      case AnyType::Kind:
      case AnyListType::Kind:
      case AnyTupleType::Kind:
      case AnyClassType::Kind:
      case AnyEnumType::Kind:
        // no op, there is nothing to tag
        break;
      case c10::SymIntType::Kind:
        // TODO: Can this really show up though? :think:
        TORCH_CHECK(!w.value.toSymInt().is_heap_allocated());
        // no op, there is nothing to tag
        break;
      case c10::SymFloatType::Kind:
        TORCH_CHECK(!w.value.toSymFloat().is_symbolic());
        // no op, there is nothing to tag
        break;
      case c10::SymBoolType::Kind:
        TORCH_CHECK(!w.value.toSymBool().is_heap_allocated());
        // no op, there is nothing to tag
        break;
      case DynamicType::Kind:
      case UnionType::Kind:
      case EnumType::Kind:
        // TODO(gmagogsfm): Implement serialization/deserialization of Enum.
        TORCH_INTERNAL_ASSERT(false);
      case TupleType::Kind: {
```
- **EN**: This chunk continues `restoreAccurateTypeTags` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `restoreAccurateTypeTags`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 109-144
```cpp
        auto t = w.value.toTuple();
        for (size_t i = 0; i < w.type->containedTypeSize(); ++i) {
          Work elem = {w.type->containedType(i), t->elements().at(i)};
          to_process.emplace_back(std::move(elem));
        }
      } break;
      case FutureType::Kind: {
        auto f = w.value.toFuture();
        if (f->completed()) {
          Work elem = {w.type->containedType(0), f->value()};
          to_process.emplace_back(std::move(elem));
        }
      } break;
      case AwaitType::Kind: {
        auto aw = w.value.toAwait();
        if (aw->completed()) {
          Work elem = {w.type->containedType(0), aw->wait()};
          to_process.emplace_back(std::move(elem));
        }
      } break;
      case OptionalType::Kind: {
        if (!w.value.isNone()) {
          Work elem = {w.type->containedType(0), w.value};
          to_process.emplace_back(std::move(elem));
        }
      } break;
      case ListType::Kind: {
        // specialized lists do not need their type refined, so we can exit
        // early here
        if (!w.value.isList()) {
          break;
        }
        auto elem_type = w.type->containedType(0);
        auto lst = w.value.toList();
        lst.unsafeSetElementType(elem_type);
        for (const IValue& item : lst) {
```
- **EN**: This chunk continues `restoreAccurateTypeTags` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `restoreAccurateTypeTags`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 145-176
```cpp
          Work elem = {elem_type, item};
          to_process.emplace_back(std::move(elem));
        }
      } break;
      case DictType::Kind: {
        auto d = w.value.toGenericDict();
        auto keyType = w.type->containedType(0);
        auto valType = w.type->containedType(1);
        d.unsafeSetKeyType(keyType);
        d.unsafeSetValueType(valType);
        for (const auto& item : d) {
          Work kelem = {keyType, item.key()};
          Work velem = {valType, item.value()};
          to_process.emplace_back(std::move(kelem));
          to_process.emplace_back(std::move(velem));
        }
      } break;
      // in both cases the dynamic type is a class, and we are going to tag with
      // the dynamic type
      case InterfaceType::Kind:
      case ClassType::Kind: {
        auto obj = w.value.toObject();
        auto typ = obj->type(); // note: intentionally using the dynamic type,
                                // the static type is potentially less accurate
        for (size_t i = 0; i < typ->numAttributes(); ++i) {
          Work elem = {typ->getAttribute(i), obj->getSlot(i)};
          to_process.emplace_back(std::move(elem));
        }
      };
    }
  }
}
```
- **EN**: This chunk continues `restoreAccurateTypeTags` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段延续了 `restoreAccurateTypeTags`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 178-204
```cpp
namespace {
template <typename T>
bool is(const Type& type) {
  if (type.kind() == T::Kind) {
    return true;
  }
  if (auto dyn = type.castRaw<c10::DynamicType>()) {
    return dyn->tag() == c10::DynamicTypeTrait<T>::tagValue();
  }
  return false;
}
} // namespace

static void restoreContainerTypeTags(
    const IValue& ivalue,
    const TypePtr& type) {
  if (is<DictType>(*type)) {
    auto dict = ivalue.toGenericDict();
    dict.unsafeSetKeyType(type->containedType(0));
    dict.unsafeSetValueType(type->containedType(1));
  } else if (is<ListType>(*type)) {
    ivalue.toList().unsafeSetElementType(type->containedType(0));
  } else {
    TORCH_CHECK(
        false, "Unknown type for tag restoration: " + type->annotation_str());
  }
}
```
- **EN**: This chunk defines `restoreContainerTypeTags`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `restoreContainerTypeTags`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 206-238
```cpp
IValue Unpickler::parse_ivalue() {
  run();
  TORCH_CHECK(
      stack_.size() == 1,
      "Unpickler expected 1 element on the stack, but found ",
      stack_.size());
  if (version_ <= 2) {
    // See [type tag serialization]
    restoreAccurateTypeTagsIfPossible(stack_[0]);
  }
  return stack_[0];
}

double Unpickler::readFloat() {
  AT_ASSERT(sizeof(double) == 8);
  double big_endian = read<double>();
#if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
  double little_endian = 0;

  // Pickle floats are big endian, so reverse the bytes
  auto big_endian_ptr = reinterpret_cast<const char*>(&big_endian);
  std::reverse_copy(
      big_endian_ptr,
      big_endian_ptr + sizeof(big_endian),
      reinterpret_cast<char*>(&little_endian));

  return little_endian;
#elif __BYTE_ORDER__ == __ORDER_BIG_ENDIAN__
  return big_endian;
#else
#error Unexpected or undefined __BYTE_ORDER__
#endif
}
```
- **EN**: This chunk defines `readFloat`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `readFloat`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 240-273
```cpp
void Unpickler::run() {
  // Expect a PROTO opcode and protocol number at the start of blob
  auto opcode = readOpCode();
  TORCH_CHECK(
      opcode == PickleOpCode::PROTO,
      "Expected PROTO opcode at the start"
      " of pickle archive, found ",
      int(static_cast<uint8_t>(opcode)));
  uint8_t protocol = read<uint8_t>();
  TORCH_CHECK(
      protocol == 2,
      "Only Pickle protocol 2 is supported, found protocol = ",
      protocol);

  while (true) {
    PickleOpCode opcode = readInstruction();
    if (opcode == PickleOpCode::STOP) {
      return;
    }
  }
}
void Unpickler::setInput(size_t memo_id) {
  AT_ASSERT(!stack_.empty());
  if (memo_id >= memo_table_.size()) {
    memo_table_.resize(memo_id + 1);
  }
  memo_table_[memo_id] = stack_.back();
}

static std::vector<int64_t> tupleToIntList(const IValue& v) {
  return fmap(v.toTupleRef().elements(), [](const IValue& v) -> int64_t {
    return v.toInt();
  });
}
```
- **EN**: This chunk defines `tupleToIntList`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `tupleToIntList`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 275-310
```cpp
// note we cannot use toIntList, toDoubleList because during unpickling the
// lists are not yet tagged
template <typename T>
static std::vector<T> convertList(const IValue& v) {
  return fmap(v.toListRef(), [](const IValue& elem) { return elem.to<T>(); });
}

PickleOpCode Unpickler::readInstruction() {
  auto opcode = readOpCode();
  switch (opcode) {
    case PickleOpCode::EMPTY_LIST: {
      stack_.emplace_back(c10::impl::GenericList(AnyType::get()));
    } break;
    case PickleOpCode::EMPTY_TUPLE: {
      if (empty_tuple_.isNone()) {
        // we only need one object, since tuples are not mutable.
        empty_tuple_ = c10::ivalue::Tuple::create(std::vector<IValue>());
      }
      stack_.emplace_back(empty_tuple_);
    } break;
    case PickleOpCode::BINPUT: {
      size_t memo_id = read<uint8_t>();
      setInput(memo_id);
    } break;
    case PickleOpCode::LONG_BINPUT: {
      TORCH_CHECK(
          std::numeric_limits<size_t>::max() >=
              std::numeric_limits<uint32_t>::max(),
          "Found a LONG_BINPUT opcode, but size_t on this system is "
          "not big enough to decode it");
      size_t memo_id = read<uint32_t>();
      setInput(memo_id);
    } break;
    case PickleOpCode::MARK: {
      // Mark location of the container ivalue in the stack
      marks_.push_back(stack_.size());
```
- **EN**: This chunk defines `readInstruction`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `readInstruction`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 311-346
```cpp
    } break;
    case PickleOpCode::NEWTRUE: {
      stack_.emplace_back(true);
    } break;
    case PickleOpCode::NEWFALSE: {
      stack_.emplace_back(false);
    } break;
    case PickleOpCode::NONE: {
      stack_.emplace_back();
    } break;
    case PickleOpCode::BININT1: {
      uint8_t value = read<uint8_t>();
      stack_.emplace_back(int64_t(value));
    } break;
    case PickleOpCode::BININT2: {
      uint16_t value = from_le16(read<uint16_t>());
      stack_.emplace_back(int64_t(value));
    } break;
    case PickleOpCode::BININT: {
      int32_t value = from_le32(read<int32_t>());
      stack_.emplace_back(int64_t(value));
    } break;
    case PickleOpCode::LONG1: {
      // Only read LONG1s with 8 as the length
      uint8_t length = read<uint8_t>();
      TORCH_CHECK(length == 8, "Expected length to be 8, got ", int(length));
      stack_.emplace_back(int64_t(from_le64(read<int64_t>())));
    } break;
    case PickleOpCode::BINUNICODE: {
      uint32_t length = from_le32(read<uint32_t>());
      stack_.emplace_back(readBytes(length));
    } break;
    case PickleOpCode::BINUNICODE8: {
      int64_t length = from_le64(read<int64_t>());
      stack_.emplace_back(readBytes(length));
    } break;
```
- **EN**: This chunk continues `readInstruction` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readInstruction`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 347-382
```cpp
    case PickleOpCode::BINFLOAT:
      stack_.emplace_back(readFloat());
      break;
    case PickleOpCode::TUPLE: {
      TORCH_CHECK(!marks_.empty(), "Parsing error: marks_ is empty");
      size_t start = marks_.back();
      marks_.pop_back();
      TORCH_CHECK(
          stack_.size() >= start,
          "Parsing error: wrong start index ",
          start,
          " for stack_ of size ",
          stack_.size());
      const auto tupleSize = stack_.size() - start;
      switch (tupleSize) {
        case 3: {
          auto e3 = pop(stack_);
          auto e2 = pop(stack_);
          auto e1 = pop(stack_);
          stack_.emplace_back(c10::ivalue::Tuple::create(
              std::move(e1), std::move(e2), std::move(e3)));
          break;
        }
        case 2: {
          auto e2 = pop(stack_);
          auto e1 = pop(stack_);
          stack_.emplace_back(
              c10::ivalue::Tuple::create(std::move(e1), std::move(e2)));
          break;
        }
        case 1:
          stack_.emplace_back(c10::ivalue::Tuple::create(pop(stack_)));
          break;
        default: {
          auto start_it = stack_.begin() + static_cast<std::ptrdiff_t>(start);
          std::vector<IValue> elements{
```
- **EN**: This chunk continues `readInstruction` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readInstruction`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 383-418
```cpp
              std::make_move_iterator(start_it),
              std::make_move_iterator(stack_.end())};
          stack_.erase(start_it, stack_.end());
          stack_.emplace_back(c10::ivalue::Tuple::create(std::move(elements)));
          break;
        }
      }
    } break;
    case PickleOpCode::TUPLE1: {
      TORCH_CHECK(
          !stack_.empty(),
          "Parsing error: stack_ contains ",
          stack_.size(),
          " elements, at least 1 expected");
      stack_.emplace_back(c10::ivalue::Tuple::create(pop(stack_)));
    } break;
    case PickleOpCode::TUPLE2: {
      TORCH_CHECK(
          stack_.size() > 1,
          "Parsing error: stack_ contains ",
          stack_.size(),
          " elements, at least 2 expected");
      auto e2 = pop(stack_);
      auto e1 = pop(stack_);
      stack_.emplace_back(
          c10::ivalue::Tuple::create(std::move(e1), std::move(e2)));
    } break;
    case PickleOpCode::TUPLE3: {
      TORCH_CHECK(
          stack_.size() > 2,
          "Parsing error: stack_ contains ",
          stack_.size(),
          " elements, at least 3 expected");
      auto e3 = pop(stack_);
      auto e2 = pop(stack_);
      auto e1 = pop(stack_);
```
- **EN**: This chunk continues `readInstruction` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readInstruction`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 419-454
```cpp
      stack_.emplace_back(c10::ivalue::Tuple::create(
          std::move(e1), std::move(e2), std::move(e3)));
    } break;
    case PickleOpCode::EMPTY_DICT:
      stack_.emplace_back(
          c10::impl::GenericDict(AnyType::get(), AnyType::get()));
      break;
    case PickleOpCode::APPENDS: {
      TORCH_CHECK(!marks_.empty(), "Parsing error: marks_ is empty");
      size_t start = marks_.back();
      TORCH_CHECK(
          start > 0 && start <= stack_.size(),
          "Parsing error: wrong start index ",
          start,
          " for stack_ of size ",
          stack_.size());
      auto list_ivalue = stack_.at(start - 1);
      readList(list_ivalue);
    } break;
    case PickleOpCode::APPEND: {
      TORCH_CHECK(
          stack_.size() >= 2, "Parsing error: missing elements in stack_.");
      auto list_ivalue = stack_.at(stack_.size() - 2);
      readListElements(list_ivalue, stack_.size() - 1);
    } break;
    case PickleOpCode::LIST: {
      IValue list_ivalue = c10::impl::GenericList(AnyType::get());
      readList(list_ivalue);
      stack_.push_back(std::move(list_ivalue));
    } break;
    case PickleOpCode::DICT: {
      TORCH_CHECK(!marks_.empty(), "Parsing error: marks_ is empty");
      size_t start = marks_.back();
      marks_.pop_back();
      TORCH_CHECK(
          stack_.size() > start,
```
- **EN**: This chunk continues `readInstruction` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readInstruction`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 455-490
```cpp
          "Parsing error: wrong start index ",
          start,
          " for stack_ which of size ",
          stack_.size());
      auto dict = c10::impl::GenericDict(AnyType::get(), AnyType::get());
      TORCH_CHECK(
          (stack_.size() - start) % 2 == 0,
          "Parsing error: stack_ is of size ",
          stack_.size(),
          " and start index is ",
          start,
          ", but stack_ is iterated by two elements at a time");
      for (size_t i = start; i < stack_.size(); i += 2) {
        dict.insert_or_assign(stack_[i], stack_[i + 1]);
      }
      stack_.erase(
          stack_.begin() + static_cast<std::ptrdiff_t>(start), stack_.end());
      stack_.emplace_back(std::move(dict));
    } break;
    case PickleOpCode::SETITEMS: {
      TORCH_CHECK(!marks_.empty(), "Parsing error: marks_ is empty");
      size_t start = marks_.back();
      marks_.pop_back();
      TORCH_CHECK(
          start > 0 && start <= stack_.size(),
          "Parsing error: wrong start index for stack_");
      auto dict = stack_.at(start - 1).toGenericDict();
      TORCH_CHECK(
          (stack_.size() - start) % 2 == 0,
          "Parsing error: stack_ is of size ",
          stack_.size(),
          " and start index is ",
          start,
          ", but stack_ is iterated by two elements at a time");
      for (size_t i = start; i < stack_.size(); i += 2) {
        dict.insert_or_assign(stack_[i], stack_[i + 1]);
```
- **EN**: This chunk continues `readInstruction` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readInstruction`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 491-526
```cpp
      }
      stack_.erase(
          stack_.begin() + static_cast<std::ptrdiff_t>(start), stack_.end());
    } break;
    case PickleOpCode::BINGET: {
      auto pos = read<uint8_t>();
      TORCH_CHECK(
          memo_table_.size() > pos,
          "Parsing error: out of bounds access at ",
          (size_t)pos,
          " to memo_table_ which is of size ",
          memo_table_.size());
      stack_.push_back(memo_table_.at(pos));
    } break;
    case PickleOpCode::LONG_BINGET: {
      auto pos = read<uint32_t>();
      TORCH_CHECK(
          memo_table_.size() > pos,
          "Parsing error: out of bounds access at ",
          (size_t)pos,
          " to memo_table_ which is of size ",
          memo_table_.size());
      stack_.push_back(memo_table_.at(pos));
    } break;
    case PickleOpCode::STOP:
      break;
    case PickleOpCode::GLOBAL: {
      // Module name, it's not needed for anything
      auto module_name = readString();
      auto class_name = readString();
      readGlobal(module_name, class_name);
    } break;
    case PickleOpCode::NEWOBJ: {
      TORCH_CHECK(!stack_.empty(), "Parsing error: stack_ is empty");
      // pop empty tuple, the actual action is stored in the globals_stack_
      stack_.pop_back();
```
- **EN**: This chunk continues `readInstruction` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readInstruction`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 527-557
```cpp
    } break;
    // because we have NEWOBJ do nothing, BUILD and REDUCE end up doing
    // the same thing
    case PickleOpCode::BUILD:
    case PickleOpCode::REDUCE: {
      // stack is: <functor_idx> <functor_arg>
      // extract <functor_idx> and remove from the stack:
      TORCH_CHECK(
          stack_.size() > 1,
          "Parsing error: stack_ contains ",
          stack_.size(),
          " elements, at least 2 expected");
      std::swap(*(stack_.end() - 2), *(stack_.end() - 1));
      size_t idx = stack_.back().toInt();
      stack_.pop_back();
      // stack is: <functor_arg>
      TORCH_CHECK(
          idx < globals_.size(),
          "Parsing error: out of bounds access to globals_");
      globals_.at(idx)();
    } break;
    case PickleOpCode::BINPERSID: {
      TORCH_CHECK(!stack_.empty(), "Parsing error: stack_ is empty");
      auto tuple = pop(stack_).toTuple();
      const auto& args = tuple->elements();
      AT_ASSERT(
          args.at(0).toStringRef() == "storage",
          "unknown PERSID key ",
          args.at(0).toStringRef());
      at::ScalarType type = args.at(1).toScalarType();
      const std::string& key = args.at(2).toStringRef();
```
- **EN**: This chunk continues `readInstruction` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readInstruction`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 559-591
```cpp
      at::Device device(args.at(3).toStringRef());
      // remap device location if it's not meta
      if (device_ && !device.is_meta()) {
        device = *device_;
      }

      at::Storage storage;
      if (storage_context_ != nullptr && storage_context_->hasStorage(key)) {
        // for torch.package logic where storage may be loaded already
        storage = storage_context_->getStorage(key);
      } else {
        int64_t numel = args.at(4).toInt();
        auto dtype = scalarTypeToTypeMeta(type);

        at::DataPtr storage_ptr;
        if (numel > 0) {
          // If there are no elements in the tensor, there's no point in
          // reading a zero (0) byte file from the input stream and paying
          // that cost.
          storage_ptr = read_record_(key);
        }

        storage = at::Storage(
            c10::Storage::use_byte_size_t(),
            numel * dtype.itemsize(),
            std::move(storage_ptr),
            /*allocator=*/nullptr,
            /*resizable=*/false); // NB: we didn't set any allocator for the
                                  // tensor
        if (storage_context_ != nullptr) {
          storage_context_->addStorage(key, storage);
        }
      }
```
- **EN**: This chunk defines `device`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `device`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 593-628
```cpp
      auto options = at::device(at::kCPU).dtype(type);
      if (use_storage_device_) {
        options = options.device(storage.device());
        device = storage.device();
      }

      at::Tensor tensor;
      if (options.backend() == c10::Backend::QuantizedCPU) {
        tensor = at::_empty_affine_quantized({}, options, 0, 0)
                     .set_(storage, 0, {}, {});
      } else {
        tensor = at::empty({0}, options).set_(storage);
      }

      if (device.is_cuda() || device.is_xpu() || device.is_meta() ||
          device.is_mtia() || device.is_hpu() || device.is_mps() ||
          device.is_privateuseone()) {
        tensor = tensor.to(device, tensor.scalar_type());
      } else if (device.type() != DeviceType::CPU) {
        TORCH_CHECK(
            false,
            "supported devices include CPU, CUDA, HPU and ",
            c10::get_privateuse1_backend(),
            " however got ",
            DeviceTypeName(device.type(), false));
      }
      stack_.emplace_back(std::move(tensor));
    } break;
    case PickleOpCode::SETITEM: {
      // At this OpCode, stack looks like
      // | Stack Bottom |
      // | ......       |
      // | Dict         | -> (stack_size - 3)
      // | Key          | -> (stack_size - 2)
      // | Value        | -> (stack_size - 1)
      TORCH_CHECK(
```
- **EN**: This chunk continues `device` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `device`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 629-658
```cpp
          stack_.size() >= 3,
          "Parsing error: stack doesn't have enough elements");

      auto stack_size = stack_.size();
      auto dict_pos = stack_size - 3;
      auto key_pos = stack_size - 2;
      auto val_pos = stack_size - 1;

      TORCH_CHECK(
          (dict_pos < stack_size) && (key_pos < stack_size) &&
              (val_pos < stack_size),
          "Parsing error: attempted out-of-bounds access while processing SETITEM opcode");

      auto dict = stack_.at(dict_pos).toGenericDict();
      dict.insert_or_assign(stack_.at(key_pos), stack_.at(val_pos));
      stack_.erase(
          stack_.begin() + static_cast<std::ptrdiff_t>(key_pos), stack_.end());
    } break;
    default: {
      TORCH_CHECK(
          false,
          "Unknown opcode for unpickling at ",
          // NOLINTNEXTLINE(performance-no-int-to-ptr)
          reinterpret_cast<void*>(opcode),
          ": ",
          int(static_cast<uint8_t>(opcode)));
    } break;
  }
  return opcode;
}
```
- **EN**: This chunk continues `device` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `device`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 660-695
```cpp
void Unpickler::readGlobal(
    const std::string& module_name,
    const std::string& class_name) {
  if (this->skip_next_read_global) {
    // See [NOTE] skip_next_read_global
    this->skip_next_read_global--;
    if (this->skip_next_read_global == 1) {
      if (module_name == "torch" && class_name == "Tensor") {
        // This is a special case when we are unpickling a subclassed tensor
        // with type torch.nn.Buffer. We didn't frequently run into this because
        // torch.nn.Buffer is introduced later in PyTorch 2 and this type IValue
        // will not be used in C++.
        rebuildTensor(false);
        stack_.emplace_back(int64_t(globals_.size() - 1));
        this->skip_next_read_global = 0;
        return;
      }
      // Pass through to the correct handler
    } else if (this->skip_next_read_global == 0) {
      // Corresponds to the type of `Tensor` being unpickled
      if (module_name != "torch" || class_name != "Tensor") {
        TORCH_WARN(
            "Trying to load a Subclassed Tensor, it will be converted to at::Tensor in C++");
      }
      stack_.emplace_back(int64_t(globals_.size() - 1));
      return;
    } else {
      TORCH_CHECK(false, "INVALID VALUES")
    }
  }
  // TODO [unpickler refactor] __main__ isn't used by the pickler anymore, this
  // is only here for bc-compatibility reasons
  if (module_name == "__main__") {
    if (class_name == "TensorID") {
      globals_.emplace_back([this] {
        auto setitem_data = stack_.back();
```
- **EN**: This chunk defines `readGlobal`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `readGlobal`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 696-731
```cpp
        stack_.pop_back();
        TORCH_INTERNAL_ASSERT(
            !tensor_table_.empty(),
            "Pickler tried to write a tensor but had no tensor table to write to");
        stack_.emplace_back(tensor_table_.at(setitem_data.toInt()));
      });
    } else if (class_name == "IntList") {
      globals_.emplace_back([this] {
        stack_.back().toList().unsafeSetElementType(IntType::get());
      });
    } else {
      TORCH_CHECK(false, "Unknown pickler class id", class_name);
    }
  } else if (module_name == "torch.jit._pickle") {
    if (class_name == "build_tensor_from_id") {
      globals_.emplace_back([this] {
        // Pop reduce arg off the stack
        auto data = stack_.back().toTupleRef().elements().at(0);
        stack_.pop_back();
        TORCH_CHECK(
            !tensor_table_.empty(),
            "Found a tensor table reference but Unpickler"
            " has no tensor table\n");
        stack_.emplace_back(tensor_table_.at(data.toInt()));
      });
    } else if (class_name == "restore_type_tag") {
      globals_.emplace_back([this] {
        auto tuple = stack_.back().toTuple();
        const auto& data = tuple->elements();
        auto type_str = data.at(1).toStringRef();
        stack_.pop_back();
        TypePtr type = nullptr;
        auto entry = type_cache_.find(type_str);
        if (entry != type_cache_.end()) {
          type = entry->second;
        } else {
```
- **EN**: This chunk continues `readGlobal` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readGlobal`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 732-767
```cpp
          if (type_resolver_ == nullptr) {
            // If we haven't injected a custom way of retrieving types from
            // names, use a barebones type parser.
            type = type_parser_(type_str);
          } else {
            type = type_resolver_(type_str).type_;
          }
          type_cache_[type_str] = type;
        }
        // TODO: Use lookahead to avoid creating the tuple and immediately
        // destroying it here
        restoreContainerTypeTags(data.at(0), type);
        stack_.emplace_back(data.at(0));
      });
    } else {
      TypePtr elem_type = nullptr;
      if (class_name == "build_intlist") {
        elem_type = IntType::get();
      } else if (class_name == "build_tensorlist") {
        elem_type = TensorType::get();
      } else if (class_name == "build_doublelist") {
        elem_type = FloatType::get();
      } else if (class_name == "build_boollist") {
        elem_type = BoolType::get();
      } else {
        TORCH_CHECK(false, "Unknown pickler class id ", class_name);
      }
      // Unpickle a list specialization (e.g. List[Tensor], List[int], ...)
      globals_.emplace_back([this, elem_type] {
        // Pop reduce arg off the stack
        auto data = stack_.back().toTupleRef().elements().at(0).toList();
        stack_.pop_back();
        data.unsafeSetElementType(elem_type);
        stack_.emplace_back(std::move(data));
      });
    }
```
- **EN**: This chunk continues `readGlobal` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readGlobal`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 768-796
```cpp
  } else if (
      module_name == "torch._utils" &&
      (class_name == "_rebuild_tensor_v2" ||
       class_name == "_rebuild_qtensor")) {
    // Unpickle a tensor
    bool quantized = class_name == "_rebuild_qtensor";
    rebuildTensor(quantized);
  } else if (
      module_name == "torch._tensor" &&
      (class_name == "_rebuild_from_type_v2")) {
    // Unpickle a Tensor with Python attributes or
    // a Subclassed Tensor.
    rebuildTensorFromTypeV2();
  } else if (
      module_name == "torch._utils" && (class_name == "_rebuild_parameter")) {
    // Unpickle a Parameter
    rebuildParameter();
  } else if (
      module_name == "torch._utils" && class_name == "_rebuild_sparse_tensor") {
    rebuildSparseTensor();
  } else if (module_name == "builtins" && class_name == "complex") {
    globals_.emplace_back([this] {
      auto tuple = pop(stack_).toTuple();
      const auto& elems = tuple->elements();
      AT_ASSERT(elems.size() == 2);
      auto complex =
          c10::complex<double>(elems.at(0).toDouble(), elems.at(1).toDouble());
      stack_.emplace_back(complex);
    });
```
- **EN**: This chunk continues `readGlobal` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `readGlobal`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 798-833
```cpp
  } else if (module_name == "collections" && class_name == "OrderedDict") {
    // collections.OrderedDict is used in tensor serialization for a tensor's
    // backward hooks (but they are not actually saved with this Pickler)
    globals_.emplace_back([this] {
      // drop the Tuple that was argument to OrderedDict, and replace it
      // with None OrderedDicts only appear in tensor deserialization and
      // their value is never used
      stack_.back() = IValue();
    });
  } else if (module_name == "torch" && class_name == "device") {
    globals_.emplace_back([this] {
      auto device_string = stack_.back().toTupleRef().elements().at(0);
      stack_.pop_back();
      stack_.emplace_back(c10::Device(device_string.toStringRef()));
    });
    stack_.emplace_back(int64_t(globals_.size() - 1));
    return;
  } else if (module_name == "torch.distributed.rpc" && class_name == "rref") {
#ifdef USE_RPC
    return rebuildRRef();
#else
    TORCH_INTERNAL_ASSERT(
        false,
        "RRef unpickling is only supported with the distributed package");
#endif
  } else if (module_name == "torch") {
    // Try to manually resolve several global enums
    // NOTE: this does not put a global into the global table,
    // like the other branches here because no REDUCE or BUILD will
    // be called on this value. Instead, we just put it on the stack
    // and return early
    std::optional<c10::ScalarType> scalar_type;
#define CHECK_SCALAR(_, name)          \
  if (class_name == #name "Storage") { \
    scalar_type = c10::k##name;        \
  }
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk continues `readGlobal` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段延续了 `readGlobal`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 834-869
```cpp
    AT_FORALL_SCALAR_TYPES_WITH_COMPLEX_AND_QINTS(CHECK_SCALAR)
#undef CHECK_SCALAR
    if (scalar_type.has_value()) {
      stack_.emplace_back(int64_t(*scalar_type));
      return;
    }

    std::optional<at::QScheme> qscheme;
    for (int i = 0; i < at::COMPILE_TIME_NUM_QSCHEMES; ++i) {
      if (class_name == toString(static_cast<at::QScheme>(i))) {
        qscheme = static_cast<at::QScheme>(i);
      }
    }
    if (qscheme.has_value()) {
      stack_.emplace_back(int64_t(*qscheme));
      return;
    }
    TORCH_CHECK(
        false,
        "Unpickler found unknown torch global, 'torch.",
        class_name,
        "'");
  } else {
    TORCH_CHECK(
        type_resolver_,
        "Unpickler found unknown type ",
        module_name,
        ".",
        class_name);
    at::StrongTypePtr type =
        type_resolver_(c10::QualifiedName(module_name, class_name));
    if (auto enum_type = type.type_->cast<c10::EnumType>()) {
      globals_.emplace_back([this, enum_type] {
        auto val = stack_.back();
        stack_.pop_back();
        for (const auto& p : enum_type->enumNamesValues()) {
```
- **EN**: This chunk continues `readGlobal` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `readGlobal`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 870-905
```cpp
          if (p.second == val) {
            auto enum_holder = c10::make_intrusive<at::ivalue::EnumHolder>(
                enum_type, p.first, p.second);
            stack_.emplace_back(std::move(enum_holder));
            return;
          }
        }
      });
    } else {
      // Otherwise, global is a class/object type.
      globals_.emplace_back([this, type] {
        auto val = stack_.back();
        stack_.pop_back();
        auto obj = obj_loader_(type, val);
        stack_.emplace_back(std::move(obj));
      });
    }
  }
  stack_.emplace_back(int64_t(globals_.size() - 1));
}

void Unpickler::rebuildSparseTensor() {
  globals_.emplace_back([this] {
    auto tup = pop(stack_).toTuple();
    const auto& elements = tup->elements();
    size_t idx = 0;
    auto layout = elements.at(idx++).toInt();
    at::Tensor result;
    switch (layout) {
      case static_cast<int>(c10::Layout::Sparse): {
        std::vector<int64_t> size = tupleToIntList(elements.at(idx++));
        bool requires_grad = elements.at(idx++).toBool();
        auto& indices_tensor = elements.at(idx++).toTensor();
        auto& values_tensor = elements.at(idx++).toTensor();
        auto options = values_tensor.options()
                           .layout(c10::Layout::Sparse)
```
- **EN**: This chunk defines `rebuildSparseTensor`, which parses source text or schema-like input into internal data structures. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `rebuildSparseTensor`，其作用是把源码文本或类 schema 输入解析为内部数据结构。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 906-936
```cpp
                           .requires_grad(requires_grad);
        result = at::_sparse_coo_tensor_unsafe(
            indices_tensor, values_tensor, size, options);
        result = autograd::make_variable(result, options.requires_grad());
        break;
      }
      case static_cast<int>(c10::Layout::SparseCsr): {
        std::vector<int64_t> size = tupleToIntList(elements.at(idx++));
        bool requires_grad = elements.at(idx++).toBool();
        auto& crow_indices = elements.at(idx++).toTensor();
        auto& col_indices = elements.at(idx++).toTensor();
        auto& values_tensor = elements.at(idx++).toTensor();
        auto options = values_tensor.options()
                           .layout(c10::Layout::SparseCsr)
                           .requires_grad(requires_grad);
        result = at::_sparse_csr_tensor_unsafe(
            crow_indices, col_indices, values_tensor, size, options);
        result =
            autograd::make_variable(std::move(result), options.requires_grad());
        break;
      }
      default:
        TORCH_CHECK(
            false,
            "Unsupported sparse tensor layout type in serialization ",
            static_cast<c10::Layout>(layout));
        break;
    }
    stack_.emplace_back(std::move(result));
  });
}
```
- **EN**: This chunk continues `rebuildSparseTensor` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `rebuildSparseTensor`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 938-973
```cpp
void Unpickler::rebuildTensor(bool quantized) {
  globals_.emplace_back([this, quantized] {
    auto tup = pop(stack_).toTuple();
    const auto& elements = tup->elements();
    size_t idx = 0;
    auto& storage_tensor = elements.at(idx++).toTensor();
    int64_t storage_offset = elements.at(idx++).toInt();
    std::vector<int64_t> size = tupleToIntList(elements.at(idx++));
    std::vector<int64_t> stride = tupleToIntList(elements.at(idx++));
    at::Tensor result;
    if (quantized) {
      auto qparams_tuple = elements.at(idx++).toTuple();
      const auto& qparams = qparams_tuple->elements();
      auto qscheme = static_cast<at::QScheme>(qparams.at(0).toInt());
      switch (qscheme) {
        case at::kPerTensorAffine: {
          double q_scale = qparams.at(1).toDouble();
          int64_t q_zero_point = qparams.at(2).toInt();
          result = at::_empty_affine_quantized(
              {0}, storage_tensor.options(), q_scale, q_zero_point);
        } break;
        case at::kPerChannelAffineFloatQParams:
        case at::kPerChannelAffine: {
          const auto& scales = qparams.at(1).toTensor();
          const auto& zero_points = qparams.at(2).toTensor();
          int64_t axis = qparams.at(3).toInt();
          result = at::_empty_per_channel_affine_quantized(
              {0}, scales, zero_points, axis, storage_tensor.options());
        } break;
        default:
          TORCH_CHECK(
              false,
              "Unsupported tensor quantization type in serialization ",
              toString(qscheme));
          break;
      }
```
- **EN**: This chunk defines `rebuildTensor`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `rebuildTensor`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 974-1001
```cpp
    } else {
      result = at::empty({0}, storage_tensor.options());
    }
    bool requires_grad = elements.at(idx++).toBool();
    idx++; // backwards hooks is empty
    at::TensorImpl* impl = result.unsafeGetTensorImpl();
    impl->set_storage_keep_dtype(storage_tensor.storage());
    impl->set_storage_offset(storage_offset);
    impl->set_sizes_and_strides(size, stride);
    result = autograd::make_variable(result, requires_grad);

    // Handle if math_bits were pickled.
    // See `args` of _reduce_ex_internal
    // for a regular tensor (final else case).
    // Tensors pickled before this patch didn't
    // have this argument for storing MathBits,
    // in that case, we do nothing.
    // NOTE: `math_bits` is the 7th arg.
    // NOTE: This is only meant for regular tensor and not quantized
    //       which also has 7 args serialized.
    if (!quantized && elements.size() == 7) {
      auto math_bits = elements.at(idx++).toGenericDict();
      torch::jit::setTensorMetadata(result, math_bits);
    }

    stack_.emplace_back(std::move(result));
  });
}
```
- **EN**: This chunk continues `rebuildTensor` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `rebuildTensor`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1003-1034
```cpp
void Unpickler::rebuildTensorFromTypeV2() {
  // [NOTE] skip_next_read_global
  // When rebuilding Tensor with Python Attr or Subclassed Tensor,
  // we receive `(func, type(self), args, state)` on stack for
  // `rebuildTensorFromTypeV2`.
  // Thus next call to readGlobal corresponds to `func` which is
  // the function to rebuild the base tensor.
  // The call after `func` to readGlobal corresponds to `type` of the
  // Tensor where we raise warning if the type is not `torch.Tensor`.
  this->skip_next_read_global = 2;
  auto curr_globals_idx = globals_.size();
  globals_.emplace_back([this, curr_globals_idx] {
    // args is a tuple with following data
    //  (function to rebuild base tensor, type of tensor,
    //   arguments to construct base tensor, Python State (as dict))
    auto args = pop(stack_).toTuple();
    size_t tup_idx = 0;
    const auto args_elems = args->elements();
    auto base_tensor_args = args_elems.at(tup_idx + 2).toTuple();
    auto py_state = args_elems.at(tup_idx + 3).toGenericDict();
    if (!py_state.empty()) {
      TORCH_WARN(
          "Loading Tensor with Python attributes will return at::Tensor with Python attributes being discarded");
    }
    // This calls the function to rebuild the
    // base tensor.
    // Eg. `rebuildTensor`, `rebuildSpareTensor`.
    stack_.emplace_back(base_tensor_args);
    globals_[curr_globals_idx + 1]();
    stack_.emplace_back(pop(stack_));
  });
}
```
- **EN**: This chunk defines `rebuildTensorFromTypeV2`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `rebuildTensorFromTypeV2`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1036-1071
```cpp
void Unpickler::rebuildParameter() {
  globals_.emplace_back([this] {
    auto args = pop(stack_).toTuple();
    size_t tup_idx = 0;
    const auto args_elems = args->elements();
    auto result = args_elems.at(tup_idx++).toTensor();
    auto requires_grad = args_elems.at(tup_idx++).toBool();
    result.requires_grad_(requires_grad);
    stack_.emplace_back(std::move(result));
  });
}

#ifdef USE_RPC
void Unpickler::rebuildRRef() {
  globals_.emplace_back([this] {
    // It is the same as how rref is unpickled in python,
    // see PyRRef::unpickle
    auto tuple = std::move(stack_.back()).toTuple();
    const auto& args = tuple->elements();
    stack_.pop_back();
    TORCH_INTERNAL_ASSERT(
        args.size() == distributed::rpc::RFD_TUPLE_SIZE,
        "Pickled RRefForkData must contain 7 numbers.");
    auto ownerId =
        static_cast<int16_t>(args.at(distributed::rpc::OWNER_IDX).toInt());
    // const reference will extend the lifetime of the temporary variable
    const auto& rrefId = distributed::rpc::RRefId(
        static_cast<int16_t>(args.at(distributed::rpc::RREFID_ON_IDX).toInt()),
        args.at(distributed::rpc::RREFID_ID_IDX).toInt());
    const auto& forkId = distributed::rpc::RRefId(
        static_cast<int16_t>(args.at(distributed::rpc::FORKID_ON_IDX).toInt()),
        args.at(distributed::rpc::FORKID_ID_IDX).toInt());
    auto parent =
        static_cast<int16_t>(args.at(distributed::rpc::PARENT_IDX).toInt());
    const auto& typeStr = static_cast<std::string>(
        args.at(distributed::rpc::TYPE_IDX).toStringRef());
```
- **EN**: This chunk defines `rebuildRRef`, which constructs derived state from the current inputs and invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `rebuildRRef`，其作用是根据当前输入和约束构建派生状态。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1072-1107
```cpp
    auto rrefForkData = distributed::rpc::RRefForkData(
        ownerId, rrefId, forkId, parent, typeStr);
    auto& ctx = distributed::rpc::RRefContext::getInstance();
    c10::intrusive_ptr<distributed::rpc::RRef> rref;
    TORCH_INTERNAL_ASSERT(
        type_resolver_ != nullptr, "type_resolver_ is nullptr.");
    at::StrongTypePtr type = type_resolver_(c10::QualifiedName(typeStr));
    rref = ctx.getOrCreateRRef(rrefForkData, type.type_);
    ctx.notifyOwnerAndParentOfFork(
        rrefForkData.forkId_, rrefForkData.parent_, rref);
    stack_.emplace_back(
        c10::static_intrusive_pointer_cast<c10::RRefInterface>(rref));
  });
  stack_.emplace_back(int64_t(globals_.size() - 1));
  return;
}
#endif

void Unpickler::readSlowWithBuffer(char* dest, size_t sz) {
  // First, read any partial from buffer (may be 0).
  // We explicitly assume that sz > buffer_remaining_,
  // and that sz is never bigger than buffer_.size().
  AT_ASSERT(sz > buffer_remaining_);
  const size_t from_old_buf = buffer_remaining_;
  if (from_old_buf != 0) {
    memcpy(dest, buffer_.data() + buffer_pos_, from_old_buf);
  }
  const size_t needed = sz - from_old_buf;
  // Full read into the buffer. The calls here all explicitly
  // assume that one buffer will be enough for any sz.
  AT_ASSERT(sz <= buffer_.size());
  buffer_remaining_ = reader_(buffer_.data(), buffer_.size());
  if (buffer_remaining_ < needed) {
    TORCH_CHECK(false, "Unexpected end of pickler archive.");
  }
  memcpy(dest + from_old_buf, buffer_.data(), needed);
```
- **EN**: This chunk defines `readSlowWithBuffer`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `readSlowWithBuffer`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1108-1143
```cpp
  buffer_pos_ = needed; // assignment (0'ed from read)
  buffer_remaining_ -= needed;
}

// Read a number of bytes from the input stream
std::string Unpickler::readBytes(size_t length) {
  std::string data;
  static const size_t kSmallString = 64;
  TORCH_CHECK(
      length <= data.max_size(),
      "Parsing error: can't read ",
      length,
      " bytes to a string");
  if (length <= buffer_remaining_) {
    // Fast-path: entirely in buffer.
    data.assign(buffer_.data() + buffer_pos_, length);
    buffer_pos_ += length;
    buffer_remaining_ -= length;
  } else if (length <= kSmallString) {
    // If the string is smallish, do a full buffer read,
    // and read out of that buffer.
    data.resize(length);
    readSlowWithBuffer(&data[0], length);
  } else {
    // Otherwise, for larger strings, read what we can from
    // the buffer, and then read directly to the destination.
    const size_t from_old_buf = buffer_remaining_;
    if (from_old_buf != 0) {
      data.reserve(length);
      data.append(buffer_.data() + buffer_pos_, from_old_buf);
    }
    data.resize(length);
    const size_t needed = length - from_old_buf;
    size_t nread = reader_(&data[from_old_buf], needed);
    if (nread != needed) {
      TORCH_CHECK(false, "Unexpected end of pickler archive.");
```
- **EN**: This chunk defines `readBytes`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `readBytes`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1144-1179
```cpp
    }
    buffer_remaining_ = 0;
    // buffer_pos_ has no meaning with buffer_remaining_ == 0.
  }
  return data;
}

void Unpickler::readListElements(IValue list_ivalue, size_t start) {
  auto num_elements = stack_.size() - start;
  auto elements = c10::ArrayRef<IValue>(stack_).slice(start);
  if (list_ivalue.isIntList()) {
    auto list = std::move(list_ivalue).toIntList();
    list.reserve(num_elements);
    for (const auto& elem : elements) {
      list.emplace_back(elem.toInt());
    }
  } else if (list_ivalue.isTensorList()) {
    auto list = std::move(list_ivalue).toTensorList();
    list.reserve(num_elements);
    for (const auto& elem : elements) {
      list.emplace_back(elem.toTensor());
    }
  } else if (list_ivalue.isDoubleList()) {
    auto list = std::move(list_ivalue).toDoubleList();
    list.reserve(num_elements);
    for (const auto& elem : elements) {
      list.emplace_back(elem.toDouble());
    }
  } else if (list_ivalue.isBoolList()) {
    auto list = std::move(list_ivalue).toBoolList();
    list.reserve(num_elements);
    for (const auto& elem : elements) {
      list.push_back(elem.toBool());
    }
  } else if (list_ivalue.isList()) {
    auto list = std::move(list_ivalue).toList();
```
- **EN**: This chunk defines `readListElements`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `readListElements`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1180-1203
```cpp
    list.reserve(num_elements);
    for (const auto& elem : elements) {
      list.emplace_back(elem);
    }
  } else {
    TORCH_CHECK(false, "Unknown IValue list kind: ", list_ivalue.tagKind());
  }
  stack_.erase(
      stack_.begin() + static_cast<std::ptrdiff_t>(start), stack_.end());
}

// Pop all the list items off of the stack and append them to the list at
// the corresponding MARK
void Unpickler::readList(IValue list_ivalue) {
  TORCH_CHECK(!marks_.empty(), "Parsing error: marks_ is empty");
  size_t start = marks_.back();
  marks_.pop_back();
  readListElements(std::move(list_ivalue), start);
}

static inline bool is_valid_python_id_char(char c) {
  return c == '_' || c == '.' || (c >= '0' && c <= '9') ||
      (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z');
}
```
- **EN**: This chunk defines `is_valid_python_id_char`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `is_valid_python_id_char`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1205-1239
```cpp
// Read a newline terminated string
std::string Unpickler::readString() {
  std::string ss;
  while (true) {
    auto* const bufferStart = buffer_.data() + buffer_pos_;
    const auto bufferLeft = buffer_.size() - buffer_pos_;
    char* const newlinePtr =
        static_cast<char*>(memchr(bufferStart, '\n', bufferLeft));
    if (newlinePtr) {
      // read up to newline and we are done.
      auto const charsRead = newlinePtr - bufferStart;
      ss.append(bufferStart, charsRead);
      buffer_remaining_ -= charsRead + 1;
      buffer_pos_ += charsRead + 1;
      break;
    } else {
      // read whole buffer, refill
      for (const char* p = bufferStart; p < bufferStart + bufferLeft; ++p) {
        // Simple check just in case there is no terminating '\n'
        TORCH_CHECK(
            is_valid_python_id_char(*p),
            "Found character '",
            int(uint8_t(*p)),
            "' in string, ",
            "strings must be qualified Python identifiers");
      }
      ss.append(bufferStart, bufferLeft);
      buffer_remaining_ = reader_(buffer_.data(), buffer_.size());
      buffer_pos_ = 0;
    }
  }
  return ss;
}

} // namespace torch::jit
```
- **EN**: This chunk defines `readString`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `readString`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Serialization pipeline**
  - EN: Implements TorchScript import/export, bytecode, source loading, and serialized model interchange.
  - CN: 实现 TorchScript 的导入/导出、字节码、源码加载以及序列化模型交换。
- **Work**
  - EN: `Work` is a central symbol declared or implemented in this file.
  - CN: `Work` 是本文件声明或实现的核心符号。
- **restoreAccurateTypeTagsIfPossible**
  - EN: `restoreAccurateTypeTagsIfPossible` is a central symbol declared or implemented in this file.
  - CN: `restoreAccurateTypeTagsIfPossible` 是本文件声明或实现的核心符号。
- **IValue data model**
  - EN: Moves values through generic boxed runtime containers used by TorchScript.
  - CN: 通过 TorchScript 使用的通用装箱运行时容器传递数值。
- **Pickle-style encoding**
  - EN: Encodes values and object graphs into compact binary representations.
  - CN: 把值和对象图编码为紧凑的二进制表示。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/distributed/rpc/rref_context.h`, `torch/csrc/jit/api/function_impl.h`, `torch/csrc/jit/mobile/type_parser.h`, `torch/csrc/jit/serialization/storage_context.h`, `torch/csrc/jit/serialization/unpickler.h`, `torch/csrc/utils/byte_order.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/ATen.h`, `ATen/core/Dict.h`
- **Standard library / 标准库**: `string`, `utility`
- **Primary symbols in this file / 本文件核心符号**: `Work`, `restoreAccurateTypeTagsIfPossible`, `restoreAccurateTypeTags`, `is`, `restoreContainerTypeTags`, `parse_ivalue`, `readFloat`, `run`
