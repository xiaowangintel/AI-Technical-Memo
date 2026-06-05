# resolver.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/resolver.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides an abstract LoRA resolver interface and a global registry for pluggable adapter sources. / 提供抽象的 LoRA 解析器接口，以及用于可插拔适配器来源的全局注册表。

## Line-by-Line Analysis / 逐行分析
### Abstract resolver contract
```python
class LoRAResolver(ABC):
    @abstractmethod
    async def resolve_lora(
        self, base_model_name: str, lora_name: str
    ) -> LoRARequest | None:
        pass
```
**EN:** `LoRAResolver` defines the async contract for locating or downloading an adapter. Implementations are expected to translate a `(base model, LoRA name)` pair into a concrete `LoRARequest`, or return `None` when nothing can be resolved.
**CN:** `LoRAResolver` 定义了定位或下载适配器的异步接口。实现类需要把 `(基础模型, LoRA 名称)` 转换成具体的 `LoRARequest`；若无法解析，则返回 `None`。

### Resolver registry
```python
@dataclass
class _LoRAResolverRegistry:
    resolvers: dict[str, LoRAResolver] = field(default_factory=dict)
```
**EN:** The registry stores named resolver instances in memory, so the rest of the system can remain agnostic about how adapters are fetched.
**CN:** 该注册表在内存中保存带名字的解析器实例，使系统其他部分无需关心适配器究竟通过哪种方式获取。

### Registration and lookup behavior
```python
if resolver_name in self.resolvers:
    logger.warning(
        "LoRA resolver %s is already registered, and will be "
        "overwritten by the new resolver instance %s.",
        resolver_name,
        resolver,
    )

if resolver_name not in self.resolvers:
    raise KeyError(
        f"LoRA resolver '{resolver_name}' not found. "
        f"Available resolvers: {list(self.resolvers.keys())}"
    )
```
**EN:** Re-registering a name is allowed but logged as a warning, making overrides explicit. Lookup raises a detailed `KeyError` that lists available resolver names, which improves debuggability.
**CN:** 允许重复注册同名解析器，但会记录 warning，让覆盖行为更明确。查找失败时会抛出包含可用名称列表的 `KeyError`，便于排查问题。

### Module-level singleton
```python
LoRAResolverRegistry = _LoRAResolverRegistry()
```
**EN:** The file exports one shared registry instance, giving the rest of the codebase a simple import-based extension point.
**CN:** 该文件导出一个共享的注册表单例，从而为代码库其余部分提供了基于导入即可使用的扩展点。

## Key Concepts / 关键概念
- Async adapter resolution / 异步适配器解析
- Registry-based extensibility / 基于注册表的可扩展性
- Explicit override warnings / 显式覆盖告警

## Dependencies / 依赖关系
- `LoRARequest` is the resolver output type / `LoRARequest` 是解析器输出类型
- `abc` and `dataclasses` implement the interface and registry / `abc` 与 `dataclasses` 实现接口和注册表
- Logging is used for duplicate registration warnings / 通过日志提示重复注册
