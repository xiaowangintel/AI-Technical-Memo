# weak.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/weak.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows. This specific file centers on `weak.py`. Key abstractions such as `_IterationGuard, WeakIdRef` shape the public surface of the file.
- **Purpose (CN) / 用途（中文）**: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。 该文件具体围绕 `weak.py` 展开。 `_IterationGuard, WeakIdRef` 等关键抽象塑造了该文件的公开接口。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
```python
# mypy: allow-untyped-defs
from __future__ import annotations

import collections.abc as _collections_abc
import weakref
from collections.abc import Mapping, MutableMapping
from weakref import ref

from torch import Tensor


WeakRef = ref


__all__ = [
    "TensorWeakRef",
    "WeakIdRef",
    "WeakIdKeyDictionary",
    "WeakTensorKeyDictionary",
]
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch:Tensor; standard-library helpers such as __future__:annotations, collections.abc, weakref, collections.abc:Mapping. The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch:Tensor；标准库辅助模块，如 __future__:annotations, collections.abc, weakref, collections.abc:Mapping。 `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。

### Lines 23-39 / 第 23-39 行
```python
# TODO: make weakref properly thread safe following
# https://github.com/python/cpython/pull/125325
class _IterationGuard:
    # This context manager registers itself in the current iterators of the
    # weak container, such as to delay all removals until the context manager
    # exits.
    # This technique should be relatively thread-safe (since sets are).

    def __init__(self, weakcontainer) -> None:
        # Don't create cycles
        self.weakcontainer = ref(weakcontainer)

    def __enter__(self):
        w = self.weakcontainer()
        if w is not None:
            w._iterating.add(self)
        return self
```
- **EN**: It introduces or extends class-level abstractions such as `_IterationGuard`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `_IterationGuard` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 41-64 / 第 41-64 行
```python
    def __exit__(self, e, t, b):
        w = self.weakcontainer()
        if w is not None:
            s = w._iterating
            s.remove(self)
            if not s:
                w._commit_removals()


# This file defines a variant of WeakKeyDictionary that overrides the hashing
# behavior of the key to use object identity, rather than the builtin
# __eq__/__hash__ functions.  This is useful for Tensor weak keys, as their
# __eq__ implementation return a Tensor (elementwise equality), which means
# you can't use them directly with the WeakKeyDictionary in standard library.
#
# Our implementation strategy is to create a wrapper weak key object, which we
# use as a key in a stock Python dictionary.  This is similar to how weakref
# implements WeakKeyDictionary, but instead of using weakref.ref as the
# wrapper, we use a custom wrapper that has different __eq__ and __hash__
# behavior.  Note that we subsequently store this weak key directly in an
# ORDINARY dictionary, since the newly constructed WeakIdKey's only use would
# be a dictionary so it would have no strong references.  Ensuring that
# only live WeakIdKeys are in the map is handled by putting finalizers on the
# original key object.
```
- **EN**: It introduces or extends class-level abstractions such as `_IterationGuard`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms.
- **CN**: 它引入或扩展了 `_IterationGuard` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。

### Lines 67-85 / 第 67-85 行
```python
# It is simpler to implement this with composition, but if we want to
# directly reuse the callback mechanism on weakref, we need the weakref
# and the key to be exactly the same object.  Reusing the callback mechanism
# minimizes the divergence between our implementation and Lib/weakref.py
#
# NB: Prefer using this when working with weakrefs of Tensors; e.g., do
# WeakIdRef(tensor) rather than weakref.ref(tensor); it handles a number of
# easy to get wrong cases transparently for you.
class WeakIdRef(weakref.ref):
    __slots__ = ["_id"]

    def __init__(self, key, callback=None) -> None:
        # Unlike stock weakref, which preserves hash semantics of the
        # original object but lazily defers hash calls until the first
        # time the user attempts to hash the weakref, we can eagerly
        # cache the id of the key as we know this is definitely the hash
        # method
        self._id = id(key)
        super().__init__(key, callback)  # type: ignore[call-arg]
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdRef`, which organize state and behavior for this subsystem.
- **CN**: 它引入或扩展了 `WeakIdRef` 等类级抽象，用于组织该子系统的状态与行为。

### Lines 87-110 / 第 87-110 行
```python
    def __call__(self):
        r = super().__call__()
        # Special logic for Tensor PyObject resurrection
        if hasattr(r, "_fix_weakref"):
            r._fix_weakref()  # type: ignore[union-attr]
        return r

    def __hash__(self):
        return self._id

    def __eq__(self, other):
        # An attractive but wrong alternate implementation is to only test if
        # the stored _ids match.  This can lead to an ABA problem if you have:
        #
        #   a1 = A()
        #   w1 = WeakIdRef(a1)
        #   del a1
        #   a2 = A()  # suppose it gets the same ID as a1
        #   w2 = WeakIdRef(a2)
        #   print(w1 == w2)
        #
        # This should be False, as a1 and a2 are unrelated (and a1 is
        # dead anyway)
        a = self()
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdRef`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeakIdRef` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 111-129 / 第 111-129 行
```python
        b = other()
        if a is not None and b is not None:
            return a is b
        return self is other


# This is the same as WeakIdRef but equality is checked using hash() rather than id.
# This will be equivalent to the one above except for classes where hash is not their id.
class _WeakHashRef(weakref.ref):
    __slots__ = ["_id"]

    def __init__(self, key, callback=None) -> None:
        # Unlike stock weakref, which preserves hash semantics of the
        # original object but lazily defers hash calls until the first
        # time the user attempts to hash the weakref, we can eagerly
        # cache the id of the key as we know this is definitely the hash
        # method
        self._id = hash(key)
        super().__init__(key, callback)  # type: ignore[call-arg]
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdRef`, `_WeakHashRef`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeakIdRef`, `_WeakHashRef` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 131-149 / 第 131-149 行
```python
    def __call__(self):
        r = super().__call__()
        # Special logic for Tensor PyObject resurrection
        if hasattr(r, "_fix_weakref"):
            r._fix_weakref()  # type: ignore[union-attr]
        return r

    def __hash__(self):
        return self._id

    def __eq__(self, other):
        # Use hash equality to determine ref equality.
        # ScriptObject implements __hash__ to return the wrapped IValue's id, so
        # this is equivalent to doing an identity comparison.
        a = self()
        b = other()
        if a is not None and b is not None:
            return hash(a) == hash(b)
        return self is other
```
- **EN**: It introduces or extends class-level abstractions such as `_WeakHashRef`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `_WeakHashRef` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 152-168 / 第 152-168 行
```python
# This is directly adapted from cpython/Lib/weakref.py
class WeakIdKeyDictionary(MutableMapping):
    def __init__(self, dict=None, ref_type=WeakIdRef) -> None:  # CHANGED
        self.data = {}

        self.ref_type = ref_type  # CHANGED

        def remove(k, selfref=ref(self)) -> None:
            self = selfref()
            if self is not None:
                if self._iterating:
                    self._pending_removals.append(k)
                else:
                    try:
                        del self.data[k]
                    except KeyError:
                        pass
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdKeyDictionary`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently.
- **CN**: 它引入或扩展了 `WeakIdKeyDictionary` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。

### Lines 170-189 / 第 170-189 行
```python
        self._remove = remove
        # A list of dead weakrefs (keys to be removed)
        self._pending_removals = []
        self._iterating = set()
        self._dirty_len = False
        if dict is not None:
            self.update(dict)

    def _commit_removals(self) -> None:
        # NOTE: We don't need to call this method before mutating the dict,
        # because a dead weakref never compares equal to a live weakref,
        # even if they happened to refer to equal objects.
        # However, it means keys may already have been removed.
        pop = self._pending_removals.pop
        d = self.data
        while True:
            try:
                key = pop()
            except IndexError:
                return
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdKeyDictionary`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeakIdKeyDictionary` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 191-213 / 第 191-213 行
```python
            try:
                del d[key]
            except KeyError:
                pass

    def _scrub_removals(self) -> None:
        d = self.data
        self._pending_removals = [k for k in self._pending_removals if k in d]
        self._dirty_len = False

    def __delitem__(self, key) -> None:
        self._dirty_len = True
        del self.data[self.ref_type(key)]  # CHANGED

    def __getitem__(self, key):
        return self.data[self.ref_type(key)]  # CHANGED

    def __len__(self) -> int:
        if self._dirty_len and self._pending_removals:
            # self._pending_removals may still contain keys which were
            # explicitly removed, we have to scrub them (see issue #21173).
            self._scrub_removals()
        return len(self.data) - len(self._pending_removals)
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdKeyDictionary`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeakIdKeyDictionary` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 215-233 / 第 215-233 行
```python
    def __repr__(self) -> str:
        return f"<{self.__class__.__name__} at {id(self):#x}>"

    def __setitem__(self, key, value) -> None:
        self.data[self.ref_type(key, self._remove)] = value  # CHANGED

    def copy(self):
        new = WeakIdKeyDictionary()
        with _IterationGuard(self):
            for key, value in self.data.items():
                o = key()
                if o is not None:
                    new[o] = value
        return new

    __copy__ = copy

    def __deepcopy__(self, memo):
        from copy import deepcopy
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdKeyDictionary`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeakIdKeyDictionary` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 235-251 / 第 235-251 行
```python
        new = self.__class__()
        with _IterationGuard(self):
            for key, value in self.data.items():
                o = key()
                if o is not None:
                    new[o] = deepcopy(value, memo)
        return new

    def get(self, key, default=None):
        return self.data.get(self.ref_type(key), default)  # CHANGED

    def __contains__(self, key) -> bool:
        try:
            wr = self.ref_type(key)  # CHANGED
        except TypeError:
            return False
        return wr in self.data
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdKeyDictionary`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeakIdKeyDictionary` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 253-273 / 第 253-273 行
```python
    def items(self):
        with _IterationGuard(self):
            for wr, value in self.data.items():
                key = wr()
                if key is not None:
                    yield key, value

    def keys(self):
        with _IterationGuard(self):
            for wr in self.data:
                obj = wr()
                if obj is not None:
                    yield obj

    __iter__ = keys

    def values(self):
        with _IterationGuard(self):
            for wr, value in self.data.items():
                if wr() is not None:
                    yield value
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdKeyDictionary`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization.
- **CN**: 它引入或扩展了 `WeakIdKeyDictionary` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。

### Lines 275-297 / 第 275-297 行
```python
    def keyrefs(self):
        """Return a list of weak references to the keys.

        The references are not guaranteed to be 'live' at the time
        they are used, so the result of calling the references needs
        to be checked before being used.  This can be used to avoid
        creating references that will cause the garbage collector to
        keep the keys around longer than needed.

        """
        return list(self.data)

    def popitem(self):
        self._dirty_len = True
        while True:
            key, value = self.data.popitem()
            o = key()
            if o is not None:
                return o, value

    # pyrefly: ignore [bad-override]
    def pop(self, key, *args):
        self._dirty_len = True
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdKeyDictionary`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeakIdKeyDictionary` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 299-318 / 第 299-318 行
```python
        return self.data.pop(self.ref_type(key), *args)  # CHANGED

    def setdefault(self, key, default=None):
        return self.data.setdefault(
            self.ref_type(key, self._remove), default
        )  # CHANGED

    def update(self, dict=None, **kwargs) -> None:  # type: ignore[override]
        d = self.data
        if dict is not None:
            if not hasattr(dict, "items"):
                dict = type({})(dict)
            for key, value in dict.items():
                d[self.ref_type(key, self._remove)] = value  # CHANGED
        if kwargs:
            self.update(kwargs)

    def __ior__(self, other):
        self.update(other)
        return self
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdKeyDictionary`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeakIdKeyDictionary` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 320-342 / 第 320-342 行
```python
    def __or__(self, other):
        if isinstance(other, _collections_abc.Mapping):
            c = self.copy()
            c.update(other)
            return c
        return NotImplemented

    def __ror__(self, other):
        if isinstance(other, _collections_abc.Mapping):
            c = self.__class__()
            c.update(other)
            c.update(self)
            return c
        return NotImplemented

    # Default Mapping equality will tests keys for equality, but
    # we want to test ids for equality
    def __eq__(self, other):
        if not isinstance(other, Mapping):
            return NotImplemented
        return {id(k): v for k, v in self.items()} == {
            id(k): v for k, v in other.items()
        }
```
- **EN**: It introduces or extends class-level abstractions such as `WeakIdKeyDictionary`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeakIdKeyDictionary` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 345-367 / 第 345-367 行
```python
# Convenience alias
WeakTensorKeyDictionary = WeakIdKeyDictionary


class TensorWeakRef:
    """Wrapper around a weak ref of a Tensor that handles the _fix_weakref() call required when unwrapping a Tensor weakref."""

    ref: WeakRef[Tensor]

    def __init__(self, tensor: Tensor) -> None:
        if not isinstance(tensor, Tensor):
            raise AssertionError(f"expected torch.Tensor, got {type(tensor)}.")
        self.ref = weakref.ref(tensor)

    def __call__(self):
        out = self.ref()
        if out is None:
            return out
        if not isinstance(out, Tensor):
            raise AssertionError(f"expected torch.Tensor, got {type(out)}.")
        # TODO, add _fix_weakref type binding
        out._fix_weakref()  # type: ignore[attr-defined]
        return out
```
- **EN**: It introduces or extends class-level abstractions such as `TensorWeakRef`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `TensorWeakRef` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows.
  - CN: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。
- **Numeric comparison**
  - EN: Helps compare reference and transformed executions to diagnose quantization effects.
  - CN: 帮助比较参考执行与转换后执行，从而诊断量化带来的影响。
- **Benchmarking**
  - EN: Focuses on repeatable measurement, result comparison, and performance diagnostics.
  - CN: 聚焦可重复的性能测量、结果比较与性能诊断。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **Hooks and callbacks**
  - EN: Coordinates callback registration and ordered execution around framework events.
  - CN: 协调回调注册以及围绕框架事件的有序执行。
- **Serialization safety**
  - EN: Keeps persisted data exchange bounded, predictable, and easier to validate.
  - CN: 让持久化数据交换保持有界、可预测且更易验证。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch:Tensor`
- **Python standard library / Python 标准库**: `__future__:annotations`, `collections.abc`, `weakref`, `collections.abc:Mapping`, `collections.abc:MutableMapping`, `weakref:ref`
- **Explicit exports / 显式导出**: `TensorWeakRef`, `WeakIdRef`, `WeakIdKeyDictionary`, `WeakTensorKeyDictionary`
- **Primary symbols / 核心符号**: `_IterationGuard`, `WeakIdRef`, `_WeakHashRef`, `WeakIdKeyDictionary`, `TensorWeakRef`
