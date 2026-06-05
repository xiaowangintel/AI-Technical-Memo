# tokenizer_manager_score_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/tokenizer_manager_score_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements tokenizer manager score mixin logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 分词器 管理器 score mixin 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4: Import runtime dependencies / 导入运行时依赖
```python
import logging
import math
from dataclasses import dataclass
from typing import Any, Dict, List, Optional, Tuple, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 6-6: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-11: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.configs.model_config import is_cross_encoding_pooler_model
from sglang.srt.managers.embed_types import PositionalEmbeds
from sglang.srt.managers.io_struct import EmbeddingReqInput, GenerateReqInput
from sglang.srt.server_args import MIS_DELIMITER_TOKEN_ID
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 13-16: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)


@dataclass(frozen=True, slots=True)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 17-25: Define class ScoreResult / 定义类 ScoreResult
```python
class ScoreResult:
    scores: List[List[float]]
    prompt_tokens: int = 0
    # Per-item pooled hidden states (pre-head transformer output).
    # CPU tensors when return_pooled_hidden_states=True; kept as tensors so
    # in-process consumers (gRPC, engine API) avoid a .tolist() round-trip.
    # The HTTP path converts to lists in serving_score.py before JSON serialization.
    # Same layout as scores: one tensor per item (not a single packed 2D tensor).
    pooled_hidden_states: Optional[List[Optional[torch.Tensor]]] = None
```
**EN:** This block declares the class `ScoreResult`. It centers on coordinating tokenizer manager score mixin behavior.
**CN:** 该代码块声明类 `ScoreResult`。它负责承载与 分词器 管理器 score mixin 相关的核心状态与行为。

### Lines 28-28: Provide supporting module logic / 提供辅助模块逻辑
```python
class TokenizerManagerScoreMixin:
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 29-64: Implement async score prompts / 实现异步score prompts
```python
    async def score_prompts(
        self,
        prompts: Union[str, List[str], List[List[int]]],
        label_token_ids: List[int],
        apply_softmax: bool = False,
        request: Optional[Any] = None,
    ) -> ScoreResult:
        """
        Score probabilities of specified token IDs after each *full prompt*.

        This is a thin wrapper over `score_request` that treats `prompts` as
        already-composed inputs (i.e., no query/item concatenation needed).

        Args:
            prompts: A single prompt string, a list of prompt strings, or a list of
                pre-tokenized prompt token ID sequences.
            label_token_ids: Token IDs to compute probabilities for.
            apply_softmax: Whether to normalize probabilities using softmax.
            request: Optional FastAPI request object.

        Returns:
            ScoreResult with:
                scores: List of score lists, one for each prompt, each in the order of label_token_ids.
                prompt_tokens: The number of prompt tokens processed.
        """
        # Text prompts
        if isinstance(prompts, str) or (
            isinstance(prompts, list) and (not prompts or isinstance(prompts[0], str))
        ):
            return await self.score_request(
                query="",
                items=prompts,  # type: ignore[arg-type]
                label_token_ids=label_token_ids,
                apply_softmax=apply_softmax,
                item_first=False,
                request=request,
```
**EN:** This block implements the async method `score_prompts(prompts, label_token_ids, apply_softmax, request)` on `TokenizerManagerScoreMixin`. It focuses on Score probabilities of specified token IDs after each *full prompt*., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的异步方法 `score_prompts(prompts, label_token_ids, apply_softmax, request)`。它围绕 `score_prompts` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 65-78: Continue async score prompts / 继续说明异步score prompts
```python
            )

        # Tokenized prompts
        if isinstance(prompts, list) and (not prompts or isinstance(prompts[0], list)):
            return await self.score_request(
                query=[],
                items=prompts,
                label_token_ids=label_token_ids,
                apply_softmax=apply_softmax,
                item_first=False,
                request=request,
            )

        raise ValueError("Invalid prompts type for score_prompts.")
```
**EN:** This block implements the async method `score_prompts(prompts, label_token_ids, apply_softmax, request)` on `TokenizerManagerScoreMixin`. It focuses on Score probabilities of specified token IDs after each *full prompt*., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的异步方法 `score_prompts(prompts, label_token_ids, apply_softmax, request)`。它围绕 `score_prompts` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 80-107: Implement build multi item token sequence / 实现build multi item Token sequence
```python
    def _build_multi_item_token_sequence(
        self, query: List[int], items: List[List[int]], delimiter_token_id: int
    ) -> Tuple[List[int], List[int]]:
        """
        Build a single token sequence for multi-item scoring.
        Format: query<delimiter>item1<delimiter>item2<delimiter>item3<delimiter>

        Args:
            query: Query token IDs
            items: List of item token ID sequences
            delimiter_token_id: Token ID to use as delimiter

        Returns:
            Tuple of (combined token sequence, delimiter indices)
        """
        combined_sequence = query[:]  # Start with query
        delimiter_indices = []

        for item in items:
            delimiter_indices.append(len(combined_sequence))
            combined_sequence.append(delimiter_token_id)  # Add delimiter
            combined_sequence.extend(item)  # Add item tokens

        # Add final delimiter after the last item for logprob extraction
        delimiter_indices.append(len(combined_sequence))
        combined_sequence.append(delimiter_token_id)

        return combined_sequence, delimiter_indices
```
**EN:** This block implements the method `_build_multi_item_token_sequence(query, items, delimiter_token_id)` on `TokenizerManagerScoreMixin`. It focuses on Build a single token sequence for multi-item scoring., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_build_multi_item_token_sequence(query, items, delimiter_token_id)`。它围绕 `_build_multi_item_token_sequence` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 109-138: Implement batch tokenize query and items / 实现批处理 tokenize query and items
```python
    def _batch_tokenize_query_and_items(
        self,
        query: Optional[Union[str, List[int]]],
        items: Optional[Union[str, List[str], List[List[int]]]],
    ) -> Tuple[List[int], List[List[int]]]:
        """
        Tokenize query and items into token IDs.

        Args:
            query: The query text (str) or pre-tokenized token IDs (List[int]).
            items: Item texts or pre-tokenized token IDs.

        Returns:
            (query_ids, items_ids): query token IDs and list of per-item token IDs.
        """
        if isinstance(query, str):
            query_ids = self.tokenizer.encode(query)
        else:
            query_ids = list(query)

        items_list = [items] if isinstance(items, str) else items

        items_ids = []
        for item in items_list:
            if isinstance(item, str):
                items_ids.append(self.tokenizer.encode(item))
            else:
                items_ids.append(list(item))

        return query_ids, items_ids
```
**EN:** This block implements the method `_batch_tokenize_query_and_items(query, items)` on `TokenizerManagerScoreMixin`. It focuses on Tokenize query and items into token IDs., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_batch_tokenize_query_and_items(query, items)`。它围绕 `_batch_tokenize_query_and_items` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 140-175: Implement process multi item scoring results / 实现process multi item scoring results
```python
    def _process_multi_item_scoring_results(
        self,
        results: Any,
        items: List,
        label_token_ids: Optional[List[int]],
        apply_softmax: bool,
        batch_request=None,
        return_pooled_hidden_states: bool = False,
    ) -> ScoreResult:
        """
        Process results from multi-item scoring request.

        Extracts per-delimiter scores from whichever field the scheduler
        populated (input_token_ids_logprobs for generation models,
        embedding for classification models), then uniformly validates,
        skips the query-boundary delimiter, and normalizes.

        Args:
            results: Results from generate_request
            items: List of items being scored
            label_token_ids: Token IDs to extract scores for
            apply_softmax: Whether to apply softmax normalization
            batch_request: The original batch request containing input sequence
            return_pooled_hidden_states: Whether to extract pooled hidden states
                from the result and include them in the ScoreResult.

        Returns:
            ScoreResult with per-item scores, prompt token count, and optional
            pooled_hidden_states (when return_pooled_hidden_states=True and the
            model populated the field).
        """
        single_result = results[0] if isinstance(results, list) else results
        meta_info = single_result.get("meta_info", {})
        num_items = len(items) if isinstance(items, list) else 1
        expected_count = num_items + 1
        request_id = meta_info.get("id", "<unknown>")
```
**EN:** This block implements the method `_process_multi_item_scoring_results(results, items, label_token_ids, apply_softmax, batch_request, ...)` on `TokenizerManagerScoreMixin`. It focuses on Process results from multi-item scoring request., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_process_multi_item_scoring_results(results, items, label_token_ids, apply_softmax, batch_request, ...)`。它围绕 `_process_multi_item_scoring_results` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 176-211: Continue process multi item scoring results / 继续说明process multi item scoring results
```python
        prompt_tokens = meta_info.get("prompt_tokens", 0)

        # Extract per-delimiter scores from whichever field has them
        input_logprobs = meta_info.get("input_token_ids_logprobs", [])
        embedding = single_result.get("embedding")

        if input_logprobs:
            # Generation model: extract label-token logprobs at each delimiter
            per_delimiter_scores = []
            for logprobs_data in input_logprobs:
                logprobs = self._extract_logprobs_for_tokens(
                    logprobs_data, label_token_ids
                )
                score_list = self._convert_logprobs_to_scores(
                    logprobs, label_token_ids, apply_softmax
                )
                per_delimiter_scores.append(score_list)
        elif embedding is not None:
            # Classification model: scores are directly in 2D embedding.
            if apply_softmax:
                scores_tensor = (
                    torch.tensor(embedding)
                    if isinstance(embedding, list)
                    else embedding
                )
                scores_tensor = torch.nn.functional.softmax(scores_tensor, dim=-1)
                per_delimiter_scores = scores_tensor.tolist()
            else:
                per_delimiter_scores = (
                    embedding if isinstance(embedding, list) else embedding.tolist()
                )
        else:
            raise RuntimeError(
                f"No scoring data found for multi-item scoring request {request_id}. "
                "Expected either input_token_ids_logprobs or embedding."
            )
```
**EN:** This block implements the method `_process_multi_item_scoring_results(results, items, label_token_ids, apply_softmax, batch_request, ...)` on `TokenizerManagerScoreMixin`. It focuses on Process results from multi-item scoring request., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_process_multi_item_scoring_results(results, items, label_token_ids, apply_softmax, batch_request, ...)`。它围绕 `_process_multi_item_scoring_results` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 212-234: Continue process multi item scoring results / 继续说明process multi item scoring results
```python

        # Validate delimiter count
        if len(per_delimiter_scores) != expected_count:
            raise RuntimeError(
                f"Expected {expected_count} delimiter entries for multi-item scoring "
                f"with {num_items} items, but got {len(per_delimiter_scores)}. "
                f"Request ID: {request_id}"
            )

        # Skip the first delimiter (query-item boundary)
        scores = per_delimiter_scores[1:]

        phs_list = None
        if return_pooled_hidden_states:
            raw_phs = single_result.get("pooled_hidden_state")
            if raw_phs is not None and len(raw_phs) == expected_count:
                phs_list = raw_phs[1:]

        return ScoreResult(
            scores=scores,
            prompt_tokens=prompt_tokens,
            pooled_hidden_states=phs_list,
        )
```
**EN:** This block implements the method `_process_multi_item_scoring_results(results, items, label_token_ids, apply_softmax, batch_request, ...)` on `TokenizerManagerScoreMixin`. It focuses on Process results from multi-item scoring request., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_process_multi_item_scoring_results(results, items, label_token_ids, apply_softmax, batch_request, ...)`。它围绕 `_process_multi_item_scoring_results` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 236-271: Implement process single item scoring results / 实现process single item scoring results
```python
    def _process_single_item_scoring_results(
        self,
        results: Any,
        label_token_ids: Optional[List[int]],
        apply_softmax: bool,
        return_pooled_hidden_states: bool = False,
    ) -> ScoreResult:
        """
        Process results from single-item scoring request.

        For generation (CausalLM) models: reads output_token_ids_logprobs.
        For non-generation (SequenceClassification) models: reads the embedding field
        which contains pooled class logits from the classification head.

        Args:
            results: Results from generate_request
            label_token_ids: Token IDs to extract scores for (generation models only)
            apply_softmax: Whether to apply softmax normalization
            return_pooled_hidden_states: Whether to extract pooled hidden states

        Returns:
            ScoreResult with per-item scores, prompt token count, and optional pooled_hidden_states.
        """
        scores = []
        phs_list = []
        has_phs = False
        prompt_tokens = 0

        is_generation = self.is_generation
        if is_generation:
            for result in results:
                # For single-item scoring, logprobs are in output_token_ids_logprobs
                output_logprobs = result["meta_info"].get(
                    "output_token_ids_logprobs", []
                )
                prompt_tokens += result["meta_info"].get("prompt_tokens", 0)
```
**EN:** This block implements the method `_process_single_item_scoring_results(results, label_token_ids, apply_softmax, return_pooled_hidden_states)` on `TokenizerManagerScoreMixin`. It focuses on Process results from single-item scoring request., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_process_single_item_scoring_results(results, label_token_ids, apply_softmax, return_pooled_hidden_states)`。它围绕 `_process_single_item_scoring_results` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 272-307: Continue process single item scoring results / 继续说明process single item scoring results
```python

                if not output_logprobs or len(output_logprobs) == 0:
                    raise RuntimeError(
                        f"output_logprobs is empty for request "
                        f"{result['meta_info'].get('id', '<unknown>')}."
                    )

                # Extract logprobs for the first (and only) position
                logprobs = self._extract_logprobs_for_tokens(
                    output_logprobs[0], label_token_ids
                )
                score_list = self._convert_logprobs_to_scores(
                    logprobs, label_token_ids, apply_softmax
                )
                scores.append(score_list)
        else:
            for result in results:
                embedding = result.get("embedding", None)
                if embedding is None:
                    raise ValueError("Embedding not found in the result.")

                prompt_tokens += result.get("meta_info", {}).get("prompt_tokens", 0)

                if apply_softmax:
                    embedding = torch.softmax(
                        torch.as_tensor(embedding), dim=-1
                    ).tolist()

                # The classification head produces per-token logits, which the pooler reduces
                # into a single vector per input. That vector is returned in the `.embeddings`
                # field — not as semantic embeddings, but as pooled classification logits.
                # The field name is reused for compatibility with the existing
                # EmbeddingPoolerOutput API.
                scores.append(embedding)

                if return_pooled_hidden_states:
```
**EN:** This block implements the method `_process_single_item_scoring_results(results, label_token_ids, apply_softmax, return_pooled_hidden_states)` on `TokenizerManagerScoreMixin`. It focuses on Process results from single-item scoring request., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_process_single_item_scoring_results(results, label_token_ids, apply_softmax, return_pooled_hidden_states)`。它围绕 `_process_single_item_scoring_results` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 308-317: Continue process single item scoring results / 继续说明process single item scoring results
```python
                    phs = result.get("pooled_hidden_state")
                    phs_list.append(phs)
                    if phs is not None:
                        has_phs = True

        return ScoreResult(
            scores=scores,
            prompt_tokens=prompt_tokens,
            pooled_hidden_states=phs_list if has_phs else None,
        )
```
**EN:** This block implements the method `_process_single_item_scoring_results(results, label_token_ids, apply_softmax, return_pooled_hidden_states)` on `TokenizerManagerScoreMixin`. It focuses on Process results from single-item scoring request., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_process_single_item_scoring_results(results, label_token_ids, apply_softmax, return_pooled_hidden_states)`。它围绕 `_process_single_item_scoring_results` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 318-322: Import runtime dependencies / 导入运行时依赖
```python

    # ------------------------------------------------------------------
    # Embed override position resolution
    # ------------------------------------------------------------------
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 323-356: Implement resolve overrides for sequence / 实现resolve overrides for sequence
```python
    def _resolve_overrides_for_sequence(
        self,
        token_ids: List[int],
        embeds: Optional[List[torch.Tensor]],
        embed_override_token_id: int,
        position_offset: int = 0,
        label: str = "input",
    ) -> Tuple[List[torch.Tensor], List[int]]:
        """Scan token_ids for placeholder occurrences and pair with embeddings.

        Args:
            token_ids: The token sequence to scan.
            embeds: Embedding tensors to place at placeholder positions (None = skip).
            embed_override_token_id: The placeholder token ID.
            position_offset: Added to each found position (for absolute coordinates).
            label: Label for error messages (e.g. "query", "items[2]").

        Returns:
            (embeds, positions) lists. Empty lists if embeds is None.
        """
        if embeds is None:
            return [], []
        positions = [
            idx + position_offset
            for idx, tok in enumerate(token_ids)
            if tok == embed_override_token_id
        ]
        if len(positions) != len(embeds):
            raise ValueError(
                f"{label} contains {len(positions)} occurrences of "
                f"embed_override_token_id={embed_override_token_id}, "
                f"but {len(embeds)} override embeddings were provided."
            )
        return embeds, positions
```
**EN:** This block implements the method `_resolve_overrides_for_sequence(token_ids, embeds, embed_override_token_id, position_offset, label)` on `TokenizerManagerScoreMixin`. It focuses on Scan token_ids for placeholder occurrences and pair with embeddings., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_resolve_overrides_for_sequence(token_ids, embeds, embed_override_token_id, position_offset, label)`。它围绕 `_resolve_overrides_for_sequence` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 358-390: Implement resolve embed overrides for request / 实现resolve embed overrides for 请求
```python
    def _resolve_embed_overrides_for_request(
        self,
        query: List[int],
        item: List[int],
        embed_override_token_id: int,
        query_embed_overrides: Optional[List[torch.Tensor]],
        item_embeds: Optional[List[torch.Tensor]],
        item_position_offset: int,
        item_label: str,
    ) -> Optional[PositionalEmbeds]:
        """Resolve embed overrides for a single query+item pair.

        Returns PositionalEmbeds if any overrides exist, None otherwise.
        """
        q_embeds, q_positions = self._resolve_overrides_for_sequence(
            query,
            query_embed_overrides,
            embed_override_token_id,
            position_offset=0,
            label="query",
        )
        i_embeds, i_positions = self._resolve_overrides_for_sequence(
            item,
            item_embeds,
            embed_override_token_id,
            position_offset=item_position_offset,
            label=item_label,
        )
        all_embeds = q_embeds + i_embeds
        all_positions = q_positions + i_positions
        if not all_embeds:
            return None
        return PositionalEmbeds(embeds=all_embeds, positions=all_positions)
```
**EN:** This block implements the method `_resolve_embed_overrides_for_request(query, item, embed_override_token_id, query_embed_overrides, item_embeds, ...)` on `TokenizerManagerScoreMixin`. It focuses on Resolve embed overrides for a single query+item pair., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_resolve_embed_overrides_for_request(query, item, embed_override_token_id, query_embed_overrides, item_embeds, ...)`。它围绕 `_resolve_embed_overrides_for_request` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 391-395: Import runtime dependencies / 导入运行时依赖
```python

    # ------------------------------------------------------------------
    # Input preparation (tokenization + input_ids construction)
    # ------------------------------------------------------------------
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 396-431: Implement build token id inputs / 实现build Token id inputs
```python
    def _build_token_id_inputs(
        self,
        query: List[int],
        items: List[List[int]],
        item_first: bool,
        use_multi_item_scoring: bool,
        embed_override_token_id: Optional[int],
        query_embed_overrides: Optional[List[torch.Tensor]],
        item_embed_overrides: Optional[List[Optional[List[torch.Tensor]]]],
    ) -> Tuple[None, List[List[int]], Optional[list], Optional[List[int]]]:
        """Build input_ids and resolve embed overrides for token-ID inputs.

        Works identically for multi-item-scoring and single-item modes — the only difference is
        how input_ids are assembled and what position offset each item gets.

        Returns:
            (text_prompts, input_ids, positional_embed_overrides, delimiter_indices)
        """
        # Both query and items are token IDs
        has_embeds = (
            query_embed_overrides is not None or item_embed_overrides is not None
        )

        # Query placeholder positions are invariant across items — resolve once.
        # (No-op returning ([], []) if has_embeds is False or query_embed_overrides is None.)
        q_embeds, q_positions = self._resolve_overrides_for_sequence(
            query,
            query_embed_overrides,
            embed_override_token_id,
            position_offset=0,
            label="query",
        )

        if use_multi_item_scoring:
            # Multi-item scoring: concatenate with placeholder delimiter token.
            # Positions are derived from item lengths (delimiter_indices), not
```
**EN:** This block implements the method `_build_token_id_inputs(query, items, item_first, use_multi_item_scoring, embed_override_token_id, ...)` on `TokenizerManagerScoreMixin`. It focuses on Build input_ids and resolve embed overrides for token-ID inputs., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_build_token_id_inputs(query, items, item_first, use_multi_item_scoring, embed_override_token_id, ...)`。它围绕 `_build_token_id_inputs` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 432-467: Continue build token id inputs / 继续说明build Token id inputs
```python
            # by scanning for this token — it exists only for FlashInfer compat.
            delimiter_token_id = MIS_DELIMITER_TOKEN_ID
            combined_input_ids, delimiter_indices = (
                self._build_multi_item_token_sequence(query, items, delimiter_token_id)
            )
            input_ids = [combined_input_ids]

            if not has_embeds:
                return None, input_ids, None, delimiter_indices

            # Resolve embed overrides across the combined multi-item-scoring sequence.
            all_embeds: List[torch.Tensor] = list(q_embeds)
            all_positions: List[int] = list(q_positions)
            current_offset = len(query) + 1  # +1 for first delimiter
            for i, item in enumerate(items):
                item_embs = item_embed_overrides[i] if item_embed_overrides else None
                i_embeds, i_positions = self._resolve_overrides_for_sequence(
                    item,
                    item_embs,
                    embed_override_token_id,
                    position_offset=current_offset,
                    label=f"items[{i}]",
                )
                all_embeds.extend(i_embeds)
                all_positions.extend(i_positions)
                current_offset += len(item) + 1  # +1 for delimiter

            if all_embeds:
                # PositionalEmbeds.__post_init__ does the single torch.cat stack.
                positional_embed_overrides = [
                    PositionalEmbeds(embeds=all_embeds, positions=all_positions)
                ]
            else:
                positional_embed_overrides = None
            return None, input_ids, positional_embed_overrides, delimiter_indices
```
**EN:** This block implements the method `_build_token_id_inputs(query, items, item_first, use_multi_item_scoring, embed_override_token_id, ...)` on `TokenizerManagerScoreMixin`. It focuses on Build input_ids and resolve embed overrides for token-ID inputs., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_build_token_id_inputs(query, items, item_first, use_multi_item_scoring, embed_override_token_id, ...)`。它围绕 `_build_token_id_inputs` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 468-503: Continue build token id inputs / 继续说明build Token id inputs
```python
        else:
            # Single-item scoring: process each item separately
            if item_first:
                input_ids = [item + query for item in items]
            else:
                input_ids = [query + item for item in items]

            if not has_embeds:
                return None, input_ids, None, None

            positional_embed_overrides = []
            any_overrides = False
            for i, item in enumerate(items):
                item_embs = item_embed_overrides[i] if item_embed_overrides else None
                i_embeds, i_positions = self._resolve_overrides_for_sequence(
                    item,
                    item_embs,
                    embed_override_token_id,
                    position_offset=len(query),
                    label=f"items[{i}]",
                )
                combined_embeds = q_embeds + i_embeds
                if combined_embeds:
                    positional_embed_overrides.append(
                        PositionalEmbeds(
                            embeds=combined_embeds,
                            positions=q_positions + i_positions,
                        )
                    )
                    any_overrides = True
                else:
                    positional_embed_overrides.append(None)

            return (
                None,
                input_ids,
```
**EN:** This block implements the method `_build_token_id_inputs(query, items, item_first, use_multi_item_scoring, embed_override_token_id, ...)` on `TokenizerManagerScoreMixin`. It focuses on Build input_ids and resolve embed overrides for token-ID inputs., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_build_token_id_inputs(query, items, item_first, use_multi_item_scoring, embed_override_token_id, ...)`。它围绕 `_build_token_id_inputs` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 504-506: Continue build token id inputs / 继续说明build Token id inputs
```python
                positional_embed_overrides if any_overrides else None,
                None,
            )
```
**EN:** This block implements the method `_build_token_id_inputs(query, items, item_first, use_multi_item_scoring, embed_override_token_id, ...)` on `TokenizerManagerScoreMixin`. It focuses on Build input_ids and resolve embed overrides for token-ID inputs., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_build_token_id_inputs(query, items, item_first, use_multi_item_scoring, embed_override_token_id, ...)`。它围绕 `_build_token_id_inputs` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 507-511: Import runtime dependencies / 导入运行时依赖
```python

    # ------------------------------------------------------------------
    # Main entry point
    # ------------------------------------------------------------------
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 512-547: Implement async score request / 实现异步score 请求
```python
    async def score_request(
        self,
        query: Optional[Union[str, List[int]]] = None,
        items: Optional[Union[str, List[str], List[List[int]]]] = None,
        label_token_ids: Optional[List[int]] = None,
        apply_softmax: bool = False,
        item_first: bool = False,
        embed_override_token_id: Optional[int] = None,
        query_embed_overrides: Optional[List[torch.Tensor]] = None,
        item_embed_overrides: Optional[List[Optional[List[torch.Tensor]]]] = None,
        request: Optional[Any] = None,
        return_pooled_hidden_states: bool = False,
    ) -> ScoreResult:
        """
        Score the probability of specified token IDs appearing after the given (query + item) pair.

        This method supports two scoring approaches:
        1. Single-Item scoring (default): Process each query+item pair independently
        2. Multi-Item scoring: When --enable-mis is set, combine query and
           multiple items into a single sequence using delimiter for efficient processing.
           Note: item_first parameter is ignored in multi-item scoring mode since it uses
           a fixed format: query<delimiter>item1<delimiter>item2<delimiter>item3<delimiter>

           Multi-item scoring works with both text and pre-tokenized inputs:
           - Text: query<delimiter_text>item1<delimiter_text>item2<delimiter_text>item3<delimiter_text>
           - Tokens: query<delimiter_token_id>item1<delimiter_token_id>item2<delimiter_token_id>item3<delimiter_token_id>

        Supports two model types:
        - Generation (CausalLM): Requires label_token_ids; returns logprob-based scores.
        - SequenceClassification: label_token_ids is optional; returns pooled class logits.

        Args:
            query: The query text or pre-tokenized query token IDs
            items: The item text(s) or pre-tokenized item token IDs
            label_token_ids: List of token IDs to compute probabilities for
            apply_softmax: Whether to normalize probabilities using softmax
```
**EN:** This block implements the async method `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)` on `TokenizerManagerScoreMixin`. It focuses on Score the probability of specified token IDs appearing after the given (query + item) pair., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的异步方法 `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)`。它围绕 `score_request` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 548-583: Continue async score request / 继续说明异步score 请求
```python
            item_first: If True, prepend items to query. Ignored for multi-item scoring.
            embed_override_token_id: Placeholder token ID for embedding override positions.
            query_embed_overrides: Embedding vectors replacing placeholder tokens in query.
            item_embed_overrides: Per-item embedding vectors replacing placeholder tokens in items.
            request: Optional FastAPI request object
            return_pooled_hidden_states: Whether to include the raw pooled transformer
                hidden states (before the task-specific head) in the result. Only
                supported for non-generation models (SequenceClassification,
                RewardModel). Raises ValueError for CausalLM models.

        Returns:
            ScoreResult with:
                scores: List of score lists, one per item.
                prompt_tokens: The number of prompt tokens processed.
                pooled_hidden_states: Per-item CPU tensors when
                    return_pooled_hidden_states=True and the model supports it;
                    None otherwise.
        """
        is_generation = self.is_generation

        if is_generation and label_token_ids is None:
            raise ValueError(
                "label_token_ids is required for generation (CausalLM) models."
            )
        if items is None:
            raise ValueError("items must be provided")
        if not items:
            return ScoreResult(scores=[], prompt_tokens=0)

        has_embeds = (
            query_embed_overrides is not None or item_embed_overrides is not None
        )
        if has_embeds and embed_override_token_id is None:
            raise ValueError(
                "embed_override_token_id is required when query_embed_overrides "
                "or item_embed_overrides are supplied."
```
**EN:** This block implements the async method `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)` on `TokenizerManagerScoreMixin`. It focuses on Score the probability of specified token IDs appearing after the given (query + item) pair., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的异步方法 `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)`。它围绕 `score_request` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 584-619: Continue async score request / 继续说明异步score 请求
```python
            )
        if item_first and has_embeds:
            raise ValueError("item_first is not supported when embeddings are supplied")
        if item_embed_overrides is not None and len(item_embed_overrides) != len(items):
            raise ValueError(
                f"item_embed_overrides length ({len(item_embed_overrides)}) "
                f"must match items length ({len(items)})."
            )
        if self.tokenizer is not None and label_token_ids is not None:
            vocab_size = self.tokenizer.vocab_size
            for token_id in label_token_ids:
                if token_id >= vocab_size:
                    raise ValueError(
                        f"Token ID {token_id} is out of vocabulary (vocab size: {vocab_size})"
                    )

        # Check if multi-item scoring is enabled
        use_multi_item_scoring = self.server_args.enable_mis

        input_ids = None
        text_prompts = None
        positional_embed_overrides = None
        delimiter_indices = None

        use_text_prompts = isinstance(query, str) and not has_embeds

        if use_text_prompts:
            # Both query and items are text
            items_list = [items] if isinstance(items, str) else items
            if use_multi_item_scoring:
                # Tokenize separately, then combine at token level with placeholder
                # delimiter. Positions come from item lengths (delimiter_indices),
                # not from scanning for this token — it's for FlashInfer compat only.
                delimiter_token_id = MIS_DELIMITER_TOKEN_ID
                query_ids, items_ids = self._batch_tokenize_query_and_items(
                    query, items_list
```
**EN:** This block implements the async method `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)` on `TokenizerManagerScoreMixin`. It focuses on Score the probability of specified token IDs appearing after the given (query + item) pair., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的异步方法 `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)`。它围绕 `score_request` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 620-655: Continue async score request / 继续说明异步score 请求
```python
                )
                combined_input_ids, delimiter_indices = (
                    self._build_multi_item_token_sequence(
                        query_ids, items_ids, delimiter_token_id
                    )
                )
                input_ids = [combined_input_ids]
            else:
                # Single-item scoring: create separate prompts for each item
                if item_first:
                    text_prompts = [f"{item}{query}" for item in items_list]
                else:
                    text_prompts = [f"{query}{item}" for item in items_list]

        elif (
            isinstance(query, list)
            and isinstance(items, list)
            and items
            and isinstance(items[0], list)
        ):
            # Both query and items are token IDs — tokenize text inputs if needed for embed overrides
            query_ids, items_ids = query, items
            _, input_ids, positional_embed_overrides, delimiter_indices = (
                self._build_token_id_inputs(
                    query_ids,
                    items_ids,
                    item_first,
                    use_multi_item_scoring,
                    embed_override_token_id,
                    query_embed_overrides,
                    item_embed_overrides,
                )
            )
        elif has_embeds:
            # Text inputs with embed overrides — need to tokenize first to resolve positions
            query_ids, items_ids = self._batch_tokenize_query_and_items(query, items)
```
**EN:** This block implements the async method `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)` on `TokenizerManagerScoreMixin`. It focuses on Score the probability of specified token IDs appearing after the given (query + item) pair., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的异步方法 `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)`。它围绕 `score_request` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 656-691: Continue async score request / 继续说明异步score 请求
```python
            _, input_ids, positional_embed_overrides, delimiter_indices = (
                self._build_token_id_inputs(
                    query_ids,
                    items_ids,
                    item_first,
                    use_multi_item_scoring,
                    embed_override_token_id,
                    query_embed_overrides,
                    item_embed_overrides,
                )
            )
        else:
            raise ValueError(
                "Invalid combination of query/items types for score_request."
            )

        if return_pooled_hidden_states:
            if is_generation:
                raise ValueError(
                    "return_pooled_hidden_states is not supported for CausalLM models. "
                    "It requires a model with a task-specific head "
                    "(e.g. SequenceClassification or RewardModel)."
                )
            model_config = self.model_config
            if model_config is not None:
                archs = getattr(model_config.hf_config, "architectures", []) or []
                if is_cross_encoding_pooler_model(archs):
                    raise ValueError(
                        f"return_pooled_hidden_states is not supported for "
                        f"{archs[0]}. This model uses CrossEncodingPooler which "
                        f"does not expose pre-head hidden states."
                    )

        # Create the appropriate request type
        mis_delimiter_indices = [delimiter_indices] if use_multi_item_scoring else None
        if is_generation:
```
**EN:** This block implements the async method `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)` on `TokenizerManagerScoreMixin`. It focuses on Score the probability of specified token IDs appearing after the given (query + item) pair., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的异步方法 `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)`。它围绕 `score_request` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 692-727: Continue async score request / 继续说明异步score 请求
```python
            batch_request = GenerateReqInput(
                text=text_prompts,
                input_ids=input_ids,
                token_ids_logprob=label_token_ids,
                return_logprob=True,
                # Set logprob_start_len=0 for multi-item scoring since we want logprobs at all delimiter positions
                logprob_start_len=0 if use_multi_item_scoring else -1,
                stream=False,
                sampling_params={"max_new_tokens": 0},
                positional_embed_overrides=positional_embed_overrides,
                multi_item_delimiter_indices=mis_delimiter_indices,
            )
        else:
            batch_request = EmbeddingReqInput(
                text=text_prompts,
                input_ids=input_ids,
                positional_embed_overrides=positional_embed_overrides,
                return_pooled_hidden_states=return_pooled_hidden_states,
                multi_item_delimiter_indices=mis_delimiter_indices,
            )

        results = await self.generate_request(batch_request, request).__anext__()

        if use_multi_item_scoring:
            # Multi-item scoring: extract scores from input_token_ids_logprobs or embedding
            return self._process_multi_item_scoring_results(
                results,
                items,
                label_token_ids,
                apply_softmax,
                batch_request,
                return_pooled_hidden_states,
            )
        else:
            # Single-item scoring: process each result separately
            return self._process_single_item_scoring_results(
```
**EN:** This block implements the async method `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)` on `TokenizerManagerScoreMixin`. It focuses on Score the probability of specified token IDs appearing after the given (query + item) pair., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的异步方法 `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)`。它围绕 `score_request` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 728-729: Continue async score request / 继续说明异步score 请求
```python
                results, label_token_ids, apply_softmax, return_pooled_hidden_states
            )
```
**EN:** This block implements the async method `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)` on `TokenizerManagerScoreMixin`. It focuses on Score the probability of specified token IDs appearing after the given (query + item) pair., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的异步方法 `score_request(query, items, label_token_ids, apply_softmax, item_first, ...)`。它围绕 `score_request` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 731-760: Implement convert logprobs to scores / 实现convert logprobs to scores
```python
    def _convert_logprobs_to_scores(
        self,
        logprobs: Dict[int, float],
        label_token_ids: List[int],
        apply_softmax: bool,
    ) -> List[float]:
        """
        Convert logprobs dictionary to ordered score list.

        Args:
            logprobs: Dictionary mapping token_id to logprob
            label_token_ids: Token IDs in desired order
            apply_softmax: Whether to apply softmax normalization

        Returns:
            List of scores in the same order as label_token_ids
        """
        score_list = [
            logprobs.get(token_id, float("-inf")) for token_id in label_token_ids
        ]

        if apply_softmax:
            score_list = torch.softmax(torch.tensor(score_list), dim=0).tolist()
        else:
            # Convert logprobs to probabilities if not using softmax
            score_list = [
                math.exp(x) if x != float("-inf") else 0.0 for x in score_list
            ]

        return score_list
```
**EN:** This block implements the method `_convert_logprobs_to_scores(logprobs, label_token_ids, apply_softmax)` on `TokenizerManagerScoreMixin`. It focuses on Convert logprobs dictionary to ordered score list., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_convert_logprobs_to_scores(logprobs, label_token_ids, apply_softmax)`。它围绕 `_convert_logprobs_to_scores` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 762-780: Implement extract logprobs for tokens / 实现extract logprobs for tokens
```python
    def _extract_logprobs_for_tokens(
        self, logprobs_data: List, label_token_ids: List[int]
    ) -> Dict[int, float]:
        """
        Extract logprobs for specified token IDs from logprobs data.

        Args:
            logprobs_data: List of (logprob, token_id, text) tuples
            label_token_ids: Token IDs to extract logprobs for

        Returns:
            Dictionary mapping token_id to logprob
        """
        logprobs = {}
        if logprobs_data:
            for logprob, token_id, _ in logprobs_data:
                if token_id in label_token_ids:
                    logprobs[token_id] = logprob
        return logprobs
```
**EN:** This block implements the method `_extract_logprobs_for_tokens(logprobs_data, label_token_ids)` on `TokenizerManagerScoreMixin`. It focuses on Extract logprobs for specified token IDs from logprobs data., so the class can advance the tokenizer manager score mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManagerScoreMixin` 上的方法 `_extract_logprobs_for_tokens(logprobs_data, label_token_ids)`。它围绕 `_extract_logprobs_for_tokens` 所承担的 分词器 管理器 score mixin 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: ScoreResult, TokenizerManagerScoreMixin
- **Domain focus / 领域焦点**: tokenizer manager score mixin / 分词器 管理器 score mixin
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, logging, math, typing
- **Third-party / 第三方库**: torch
- **Local Modules / 本地模块**: sglang.srt.configs.model_config, sglang.srt.managers.embed_types, sglang.srt.managers.io_struct, sglang.srt.server_args
