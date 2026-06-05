# pooling_params.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/pooling_params.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: EN: Normalizes and validates request parameters for pooling-style models such as embedding, classification, token-level pooling, and late interaction. / CN: 对 embedding、分类、token 级 pooling 以及 late interaction 等 pooling 类模型的请求参数进行归一化与校验。

## Line-by-Line Analysis / 逐行分析

### LateInteractionParams metadata
```python
class LateInteractionParams(
    msgspec.Struct,
    omit_defaults=True,  # type: ignore[call-arg]
    array_like=True,
):  # type: ignore[call-arg]
    """Metadata for worker-side late-interaction scoring.

    Attributes:
        mode:
            - "cache_query": cache query token embeddings
            - "score_doc": score a document against a cached query.
        query_key: stable key used for both DP routing and worker cache lookup.
        query_uses: expected number of document requests
    """

    mode: str
    query_key: str
    query_uses: int | None = None
```
**EN:** `LateInteractionParams` is a compact worker-facing metadata object for late-interaction retrieval. It tells the worker whether to cache a query or score a document, and uses `query_key`/`query_uses` to coordinate reuse across routing and cache lookup.
**CN:** `LateInteractionParams` 是面向 worker 的紧凑元数据对象，用于 late-interaction 检索。它告诉 worker 当前是在缓存 query 还是在给文档打分，并通过 `query_key`/`query_uses` 协调路由与缓存复用。

### PoolingParams fields
```python
class PoolingParams(
    msgspec.Struct,
    omit_defaults=True,  # type: ignore[call-arg]
    array_like=True,
):  # type: ignore[call-arg]
    """API parameters for pooling models.

    Attributes:
        use_activation: Whether to apply activation function to the pooler outputs.
            `None` uses the pooler's default, which is `True` in most cases.
        dimensions: Reduce the dimensions of embeddings
            if model support matryoshka representation.
    """

    # --8<-- [start:common-pooling-params]
    use_activation: bool | None = None
    # --8<-- [end:common-pooling-params]

    ## for embeddings models
    # --8<-- [start:embed-pooling-params]
    dimensions: int | None = None
    # --8<-- [end:embed-pooling-params]

    ## for step pooling models
    step_tag_id: int | None = None
    returned_token_ids: list[int] | None = None

    ## Internal use only
    task: PoolingTask | None = None
    requires_token_ids: bool = False
    skip_reading_prefix_cache: bool | None = None
    late_interaction_params: LateInteractionParams | None = None
    extra_kwargs: dict[str, Any] | None = None
    output_kind: RequestOutputKind = RequestOutputKind.FINAL_ONLY
```
**EN:** `PoolingParams` mixes user-visible controls (`use_activation`, `dimensions`) with task-routing and engine-internal fields such as step-pooling token selection, prefix-cache behavior, late-interaction metadata, extra kwargs, and output mode.
**CN:** `PoolingParams` 同时包含用户可见的控制项（如 `use_activation`、`dimensions`）以及任务路由和引擎内部字段，比如 step pooling 的 token 选择、prefix cache 行为、late interaction 元数据、额外参数和输出模式。

### Parameter catalogs and verify entry point
```python
    @property
    def all_parameters(self) -> list[str]:
        return ["dimensions", "use_activation"]

    @property
    def valid_parameters(self):
        return {
            "embed": ["dimensions", "use_activation"],
            "classify": ["use_activation"],
            "token_embed": ["dimensions", "use_activation"],
            "token_classify": ["use_activation"],
        }

    def clone(self) -> "PoolingParams":
        """Returns a deep copy of the PoolingParams instance."""
        return deepcopy(self)

    def verify(self, model_config: ModelConfig) -> None:
        # plugin task uses io_processor.parse_request to verify inputs,
        # skipping PoolingParams verify
        if self.task == "plugin":
            if self.skip_reading_prefix_cache is None:
                self.skip_reading_prefix_cache = True
            return

        # skipping verify, let plugins configure and validate pooling params
        if self.task not in self.valid_parameters:
            return

        # NOTE: Task validation needs to done against the model instance,
        # which is not available in model config. So, it's not included
        # in this method
        self._merge_default_parameters(model_config)
        self._set_default_parameters(model_config)
        self._verify_valid_parameters()
```
**EN:** `all_parameters` enumerates the user-tunable knobs, while `valid_parameters` narrows them per task. `verify()` skips plugin tasks, merges model defaults, sets missing derived values, and finally checks that the request did not specify fields the selected task cannot honor.
**CN:** `all_parameters` 列出所有面向用户的可调参数，`valid_parameters` 则按任务收窄允许集合。`verify()` 会跳过 plugin 任务，对常规任务先合并模型默认值、补齐派生默认项，最后检查请求是否携带了该任务根本不支持的字段。

### Default merge and step-pooling checks
```python
    def _merge_default_parameters(self, model_config: ModelConfig) -> None:
        pooler_config = model_config.pooler_config
        if pooler_config is None:
            return

        assert self.task is not None, "task must be set"
        valid_parameters = self.valid_parameters[self.task]

        for k in valid_parameters:
            if getattr(pooler_config, k, None) is None:
                continue

            if getattr(self, k, None) is None:
                setattr(self, k, getattr(pooler_config, k))

        if self.skip_reading_prefix_cache is None:
            # If prefix caching is enabled,
            # the output of all pooling may less than n_prompt_tokens,
            # we need to skip reading cache at this request.
            if self.task in ["token_embed", "token_classify"]:
                self.skip_reading_prefix_cache = True
            else:
                self.skip_reading_prefix_cache = False

        self._verify_step_pooling(pooler_config, valid_parameters)

    def _verify_step_pooling(
        self,
        pooler_config: PoolerConfig,
        valid_parameters: list[str],
    ):
        step_pooling_parameters = ["step_tag_id", "returned_token_ids"]
        if pooler_config.tok_pooling_type != "STEP":
            invalid_parameters = []
            for k in step_pooling_parameters:
                if getattr(self, k, None) is not None:
                    invalid_parameters.append(k)

            if invalid_parameters:
                raise ValueError(
                    f"Task {self.task} only supports {valid_parameters} "
                    f"parameters, does not support "
                    f"{invalid_parameters} parameters"
                )
        else:
            for k in step_pooling_parameters:
                if getattr(pooler_config, k, None) is None:
                    continue

                if getattr(self, k, None) is None:
                    setattr(self, k, getattr(pooler_config, k))
```
**EN:** `_merge_default_parameters()` backfills unset request fields from `pooler_config` and decides when prefix-cache reads must be disabled, especially for token-level outputs whose shape depends on uncached token positions. `_verify_step_pooling()` either forbids step-only parameters on non-STEP poolers or inherits them from model config when STEP pooling is enabled.
**CN:** `_merge_default_parameters()` 会从 `pooler_config` 回填请求中未设置的字段，并判断何时必须禁用 prefix cache 读取，尤其是对输出长度依赖未缓存 token 位置的 token 级任务。`_verify_step_pooling()` 则负责两件事：对非 STEP pooler 禁止使用 step 专属参数；对 STEP pooler 则在请求未显式给出时从模型配置继承这些参数。

### Task-specific defaults
```python
    def _set_default_parameters(self, model_config: ModelConfig):
        if self.task in ["embed", "token_embed"]:
            if self.use_activation is None:
                self.use_activation = True

            if self.dimensions is not None:
                if not model_config.is_matryoshka:
                    raise ValueError(
                        f'Model "{model_config.served_model_name}" does not '
                        f"support matryoshka representation, "
                        f"changing output dimensions will lead to poor results."
                    )

                mds = model_config.matryoshka_dimensions
                if mds is not None:
                    if self.dimensions not in mds:
                        raise ValueError(
                            f"Model {model_config.served_model_name!r} "
                            f"only supports {str(mds)} matryoshka dimensions, "
                            f"use other output dimensions will "
                            f"lead to poor results."
                        )
                elif self.dimensions < 1:
                    raise ValueError("Dimensions must be greater than 0")

        elif self.task in ["classify", "token_classify"]:
            if self.use_activation is None:
                self.use_activation = True
        else:
            raise ValueError(f"Unknown pooling task: {self.task!r}")
```
**EN:** Embedding-style tasks default to applying the activation function and optionally validate requested `dimensions` against the model's matryoshka capability. Classification-style tasks only manage activation defaults; anything else is treated as an unsupported pooling task.
**CN:** embedding 风格任务默认会启用激活函数，并在用户指定 `dimensions` 时检查模型是否支持对应的 matryoshka 能力。classification 风格任务只处理激活相关默认值；其它任务类型则会被视为不受支持。

### Final validation and invariants
```python
    def _verify_valid_parameters(self):
        assert self.task is not None, "task must be set"
        valid_parameters = self.valid_parameters[self.task]
        invalid_parameters = []
        for k in self.all_parameters:
            if k in valid_parameters:
                continue

            if getattr(self, k, None) is not None:
                invalid_parameters.append(k)

        if invalid_parameters:
            raise ValueError(
                f"Task {self.task!r} only supports {valid_parameters} "
                f"parameters, does not support "
                f"{invalid_parameters} parameters"
            )

    def __repr__(self) -> str:
        return (
            f"PoolingParams("
            f"task={self.task}, "
            f"dimensions={self.dimensions}, "
            f"use_activation={self.use_activation}, "
            f"step_tag_id={self.step_tag_id}, "
            f"returned_token_ids={self.returned_token_ids}, "
            f"requires_token_ids={self.requires_token_ids}, "
            f"skip_reading_prefix_cache={self.skip_reading_prefix_cache}, "
            f"late_interaction_params={self.late_interaction_params}, "
            f"extra_kwargs={self.extra_kwargs})"
        )

    def __post_init__(self) -> None:
        assert self.output_kind == RequestOutputKind.FINAL_ONLY, (
            "For pooling output_kind has to be FINAL_ONLY"
        )
```
**EN:** `_verify_valid_parameters()` scans for any user-specified field that falls outside the task's allowlist and reports it explicitly. `__post_init__()` then enforces an important invariant: pooling requests only support `FINAL_ONLY`, because there is no token-by-token stream to expose.
**CN:** `_verify_valid_parameters()` 会扫描所有用户显式设置的字段，只要它不在当前任务的允许列表里，就会明确报错。随后 `__post_init__()` 进一步保证一个关键不变量：pooling 请求只能使用 `FINAL_ONLY`，因为这类任务没有逐 token 的中间流式结果可返回。

## Key Concepts / 关键概念
- **Task-aware validation** — EN: The acceptable parameter set depends on whether the request is embedding, classification, or token-level pooling. / CN: 参数是否合法取决于任务类型，是 embedding、classification 还是 token 级 pooling。
- **Model-default merging** — EN: Request params inherit unset values from the model's pooler config before validation. / CN: 请求参数会在校验前先从模型的 pooler 配置中继承未显式设置的默认值。
- **Prefix-cache safety** — EN: Token-level pooling may disable prefix-cache reads so output shapes stay correct. / CN: token 级 pooling 可能会禁用 prefix cache 读取，以保证输出形状正确。

## Dependencies / 依赖关系
- **vllm.config.ModelConfig / PoolerConfig** — EN: Supplies model capabilities and pooler defaults used during verification. / CN: 提供校验过程中需要用到的模型能力信息和 pooler 默认配置。
- **vllm.tasks.PoolingTask** — EN: Defines the task labels that decide which parameters are valid. / CN: 定义决定参数合法性的 pooling 任务标签。
- **vllm.sampling_params.RequestOutputKind** — EN: Reused to enforce that pooling outputs are final-only. / CN: 复用该枚举来约束 pooling 输出必须是最终结果模式。
