# tasks.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tasks.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: EN: Centralizes the literal task names and task-family mappings used across request parsing and model capability checks. / CN: 集中定义请求解析和模型能力检查中使用的任务字面量名称及任务家族映射。

## Line-by-Line Analysis / 逐行分析

### Generation and pooling task literals
```python
GenerationTask = Literal["generate", "transcription", "realtime"]
GENERATION_TASKS: tuple[GenerationTask, ...] = get_args(GenerationTask)

PoolingTask = Literal[
    "embed",
    "classify",
    "token_embed",
    "token_classify",
    "plugin",
    "embed&token_classify",
]
POOLING_TASKS: tuple[PoolingTask, ...] = get_args(PoolingTask)
```
**EN:** The file encodes supported generation and pooling task names as `Literal` types, then immediately derives tuple constants with `get_args()`. This pattern keeps the type checker, runtime validators, and CLI/help surfaces anchored to the same source of truth.
**CN:** 文件把支持的 generation 与 pooling 任务名定义为 `Literal` 类型，然后立刻用 `get_args()` 派生出对应的元组常量。这样一来，类型检查器、运行时校验器以及 CLI/帮助信息都能共享同一个事实来源。

### Score-type mapping and unified task union
```python
ScoreType = Literal["bi-encoder", "cross-encoder", "late-interaction"]
SCORE_TYPE_MAP: dict[PoolingTask, ScoreType] = {
    "embed": "bi-encoder",
    "classify": "cross-encoder",
    "token_embed": "late-interaction",
}

FrontendTask = Literal["render"]
FRONTEND_TASKS: tuple[FrontendTask, ...] = get_args(FrontendTask)

SupportedTask = Literal[GenerationTask, PoolingTask, FrontendTask]
```
**EN:** `SCORE_TYPE_MAP` maps selected pooling tasks to the scoring algorithm family they imply, such as bi-encoder versus cross-encoder. Frontend-only tasks are tracked separately, and `SupportedTask` forms the union consumed by higher-level request parsing and capability gating code.
**CN:** `SCORE_TYPE_MAP` 把部分 pooling 任务映射到其对应的打分算法家族，例如 bi-encoder 或 cross-encoder。frontend 专属任务被单独维护，而 `SupportedTask` 则把这些任务集合合并成上层请求解析和能力约束逻辑会使用的统一联合类型。

## Key Concepts / 关键概念
- **Literal-driven API surface** — EN: Task names are first-class type information, not just loose strings. / CN: 任务名被当成一等类型信息管理，而不是随意散落的字符串。
- **Runtime reflection from types** — EN: `get_args()` converts type definitions into runtime tuples for validation and iteration. / CN: `get_args()` 会把类型定义反射成运行时元组，用于校验和枚举。
- **Task families** — EN: Generation, pooling, scoring, and frontend tasks are kept distinct but composable. / CN: generation、pooling、scoring 与 frontend 任务既彼此区分，又可以组合使用。

## Dependencies / 依赖关系
- **typing.Literal** — EN: Provides the closed set of legal task strings. / CN: 提供合法任务字符串的封闭集合。
- **typing.get_args** — EN: Extracts runtime tuples from the `Literal` definitions. / CN: 从 `Literal` 定义中提取运行时可用的元组。
- **Higher-level request parsing** — EN: Other modules import these aliases to validate requested task names consistently. / CN: 上层请求解析模块会导入这些别名，以一致方式校验任务名。
