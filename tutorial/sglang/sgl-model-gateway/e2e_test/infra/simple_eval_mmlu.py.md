# simple_eval_mmlu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/infra/simple_eval_mmlu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises simple eval mmlu behavior in the end-to-end infrastructure layer. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端测试基础设施 中与 simple eval mmlu 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module header and comments
```python
# Adapted from https://github.com/openai/simple-evals/
```
**EN:** These leading lines typically contain a shebang, encoding note, or comments that frame the rest of the file.
**CN:** 这些起始行通常包含 shebang、编码说明或注释，用于为后续代码提供背景。

### Lines 2-5: Module docstring
```python
"""
MMLU Evaluation - Measuring Massive Multitask Language Understanding
Dan Hendrycks et al. https://arxiv.org/abs/2009.03300
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 7-23: Imports and dependencies
```python
from __future__ import annotations

import random
import re
from typing import TYPE_CHECKING

import pandas

from . import simple_eval_common as common
from .simple_eval_common import (
    ANSWER_PATTERN_MULTICHOICE,
    HTML_JINJA,
    Eval,
    EvalResult,
    SingleEvalResult,
    format_multichoice_question,
)
```
**EN:** This block imports `__future__`, `random`, `re`, `typing`, and 3 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 25-26: If block
```python
if TYPE_CHECKING:
    from .simple_eval_common import SamplerBase
```
**EN:** This top-level block contributes supporting logic to the module and is executed or defined as part of file initialization.
**CN:** 这个顶层代码块为模块提供辅助逻辑，并在文件初始化阶段被执行或定义。

### Lines 27-89: Module constants and configuration
```python

# MMLU dataset URL (hosted by OpenAI)
MMLU_DATASET_URL = "https://openaipublic.blob.core.windows.net/simple-evals/mmlu.csv"

SUBJECT_TO_CATEGORY = {
    "abstract_algebra": "stem",
    "anatomy": "other",
    "astronomy": "stem",
    "business_ethics": "other",
    "clinical_knowledge": "other",
    "college_biology": "stem",
    "college_chemistry": "stem",
    "college_computer_science": "stem",
    "college_mathematics": "stem",
    "college_medicine": "other",
    "college_physics": "stem",
    "computer_security": "stem",
    "conceptual_physics": "stem",
    "econometrics": "social_sciences",
    "electrical_engineering": "stem",
    "elementary_mathematics": "stem",
    "formal_logic": "humanities",
    "global_facts": "other",
    "high_school_biology": "stem",
    "high_school_chemistry": "stem",
    "high_school_computer_science": "stem",
    "high_school_european_history": "humanities",
    "high_school_geography": "social_sciences",
    "high_school_government_and_politics": "social_sciences",
    "high_school_macroeconomics": "social_sciences",
    "high_school_mathematics": "stem",
    "high_school_microeconomics": "social_sciences",
    "high_school_physics": "stem",
    "high_school_psychology": "social_sciences",
    "high_school_statistics": "stem",
    "high_school_us_history": "humanities",
    "high_school_world_history": "humanities",
    "human_aging": "other",
    "human_sexuality": "social_sciences",
    "international_law": "humanities",
    "jurisprudence": "humanities",
    "logical_fallacies": "humanities",
    "machine_learning": "stem",
    "management": "other",
    "marketing": "other",
    "medical_genetics": "other",
    "miscellaneous": "other",
    "moral_disputes": "humanities",
    "moral_scenarios": "humanities",
    "nutrition": "other",
    "philosophy": "humanities",
    "prehistory": "humanities",
    "professional_accounting": "other",
    "professional_law": "humanities",
    "professional_medicine": "other",
    "professional_psychology": "social_sciences",
    "public_relations": "social_sciences",
    "security_studies": "social_sciences",
    "sociology": "social_sciences",
    "us_foreign_policy": "social_sciences",
    "virology": "other",
    "world_religions": "humanities",
}
```
**EN:** This section defines module-level names such as `MMLU_DATASET_URL`, `SUBJECT_TO_CATEGORY`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 92-132: Class `MMLUEval`
```python
class MMLUEval(Eval):
    """MMLU benchmark evaluation."""

    def __init__(self, filename: str, num_examples: int | None, num_threads: int):
        if "://" in filename:
            df = pandas.read_csv(filename, storage_options={"timeout": 30})
        else:
            df = pandas.read_csv(filename)
        examples = [row.to_dict() for _, row in df.iterrows()]
        if num_examples:
            examples = random.Random(0).sample(examples, num_examples)
        self.examples = examples
        self.num_threads = num_threads

    def __call__(self, sampler: "SamplerBase") -> EvalResult:
        def fn(row: dict) -> SingleEvalResult:
            prompt_messages = [
                sampler._pack_message(
                    content=format_multichoice_question(row), role="user"
                )
            ]
            response_text = sampler(prompt_messages)
            response_text = response_text or ""
            match = re.search(ANSWER_PATTERN_MULTICHOICE, response_text)
            extracted_answer = match.group(1) if match else None
            score = 1.0 if extracted_answer == row["Answer"] else 0.0
            html = common.jinja_env.from_string(HTML_JINJA).render(
                prompt_messages=prompt_messages,
                next_message=dict(content=response_text, role="assistant"),
                score=score,
                correct_answer=row["Answer"],
                extracted_answer=extracted_answer,
            )
            convo = prompt_messages + [dict(content=response_text, role="assistant")]
            category = SUBJECT_TO_CATEGORY.get(row["Subject"], "other")
            return SingleEvalResult(
                html=html, score=score, metrics={category: score}, convo=convo
            )

        results = common.map_with_progress(fn, self.examples, self.num_threads)
        return common.aggregate_results(results)
```
**EN:** Class `MMLUEval` groups related state and behavior. It exposes 2 method(s) that implement the module's primary abstraction.
**CN:** 类 `MMLUEval` 将相关状态与行为封装在一起，提供 2 个方法来实现本模块的核心抽象。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Test infrastructure orchestration / 测试基础设施编排
- Primary classes: `MMLUEval` / 主要类：`MMLUEval`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `random`, `re`, `typing`
- **Third-party / 第三方**: `pandas`
- **Internal / 内部模块**: `.`, `.simple_eval_common`
