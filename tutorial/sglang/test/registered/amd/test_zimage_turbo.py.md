# test_zimage_turbo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/test_zimage_turbo.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on amd zimage turbo in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 amd zimage turbo 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Document the module
```python
"""AMD nightly test for Z-Image-Turbo diffusion model (text-to-image)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 3-22: Import dependencies
```python
import io
import logging
import os

import pytest

from sglang.multimodal_gen.test.server.test_server_common import (  # noqa: F401
    DiffusionServerBase,
    diffusion_server,
)
from sglang.multimodal_gen.test.server.test_server_utils import (
    ServerContext,
    get_generate_fn,
)
from sglang.multimodal_gen.test.server.testcase_configs import (
    DiffusionSamplingParams,
    DiffusionServerArgs,
    DiffusionTestCase,
)
from sglang.test.ci.ci_register import register_amd_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 24-24: Define module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 26-26: Register CI metadata
```python
register_amd_ci(est_time=1800, suite="nightly-amd-1-gpu-zimage-turbo", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 28-39: Define module constants
```python
AMD_ZIMAGE_CASES = [
    DiffusionTestCase(
        "zimage_image_t2i",
        DiffusionServerArgs(model_path="Tongyi-MAI/Z-Image-Turbo", modality="image"),
        DiffusionSamplingParams(
            prompt="Doraemon is eating dorayaki",
            output_size="1024x1024",
        ),
    ),
]

CLIP_SCORE_THRESHOLD = 0.20
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 42-44: Define module constants
```python
ARTIFACT_DIR = os.environ.get(
    "SGLANG_DIFFUSION_ARTIFACT_DIR", "/tmp/diffusion-artifacts"
)
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 47-77: Define helper: save image and write summary
```python
def _save_image_and_write_summary(
    case_id: str, prompt: str, image_bytes: bytes, clip_score: float | None = None
):
    """Save generated image to artifact dir and write summary."""
    ext = "jpg" if image_bytes[:2] == b"\xff\xd8" else "png"
    os.makedirs(ARTIFACT_DIR, exist_ok=True)
    img_path = os.path.join(ARTIFACT_DIR, f"{case_id}.{ext}")
    with open(img_path, "wb") as f:
        f.write(image_bytes)
    logger.info("Saved image artifact: %s (%d bytes)", img_path, len(image_bytes))

    summary_file = os.environ.get("GITHUB_STEP_SUMMARY")
    if not summary_file:
        return

    clip_line = ""
    if clip_score is not None:
        status = "PASS" if clip_score >= CLIP_SCORE_THRESHOLD else "FAIL"
        clip_line = f"| CLIP Score | {clip_score:.4f} ({status}, threshold: {CLIP_SCORE_THRESHOLD}) |\n"

    md = (
        f"### Z-Image-Turbo — `{case_id}`\n\n"
        f"| | |\n|---|---|\n"
        f"| Prompt | {prompt} |\n"
        f"| Size | {len(image_bytes):,} bytes |\n"
        f"{clip_line}"
        f"| Artifact | `{case_id}.{ext}` (download from Artifacts section above) |\n\n"
    )

    with open(summary_file, "a") as f:
        f.write(md)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 80-103: Define helper: compute clip score
```python
def _compute_clip_score(image_bytes: bytes, prompt: str) -> float | None:
    """Compute CLIP cosine similarity between the image and prompt."""
    try:
        import torch
        from PIL import Image
        from transformers import CLIPModel, CLIPProcessor

        model_name = "openai/clip-vit-base-patch32"
        processor = CLIPProcessor.from_pretrained(model_name)
        model = CLIPModel.from_pretrained(model_name)
        model.eval()

        image = Image.open(io.BytesIO(image_bytes)).convert("RGB")
        inputs = processor(text=[prompt], images=image, return_tensors="pt")

        with torch.no_grad():
            outputs = model(**inputs)
            score = outputs.logits_per_image.item() / 100.0

        logger.info("CLIP score for '%s': %.4f", prompt, score)
        return score
    except Exception as e:
        logger.warning("CLIP score computation failed: %s", e)
        return None
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 106-106: Define class TestZImageTurboAMD
```python
class TestZImageTurboAMD(DiffusionServerBase):
```
**EN:** This declaration introduces the `TestZImageTurboAMD` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestZImageTurboAMD` 测试类，并说明它通过继承承担的职责。

### Lines 107-107: Document the class `TestZImageTurboAMD`
```python
    """AMD nightly test for Z-Image-Turbo text-to-image generation."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestZImageTurboAMD`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestZImageTurboAMD`的设计意图。

### Lines 109-114: Define helper: teardown class
```python
    @classmethod
    def teardown_class(cls):
        try:
            super().teardown_class()
        except AttributeError:
            pass
```
**EN:** This helper function encapsulates reusable logic inside `TestZImageTurboAMD` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestZImageTurboAMD` 内部调用，从而让场景结构更清晰。

### Lines 116-118: Define helper: case
```python
    @pytest.fixture(params=AMD_ZIMAGE_CASES, ids=lambda c: c.id)
    def case(self, request) -> DiffusionTestCase:
        return request.param
```
**EN:** This helper function encapsulates reusable logic inside `TestZImageTurboAMD` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestZImageTurboAMD` 内部调用，从而让场景结构更清晰。

### Lines 120-150: Run test: diffusion generation
```python
    def test_diffusion_generation(
        self,
        case: DiffusionTestCase,
        diffusion_server: ServerContext,
    ):
        generate_fn = get_generate_fn(
            model_path=case.server_args.model_path,
            modality=case.server_args.modality,
            sampling_params=case.sampling_params,
        )

        perf_record, content = self.run_and_collect(
            diffusion_server, case.id, generate_fn
        )

        self._validate_and_record(case, perf_record)
        self._test_v1_models_endpoint(diffusion_server, case)

        prompt = case.sampling_params.prompt or ""
        clip_score = _compute_clip_score(content, prompt)

        if clip_score is not None:
            logger.info(
                "CLIP score: %.4f (threshold: %.2f)", clip_score, CLIP_SCORE_THRESHOLD
            )
            assert clip_score >= CLIP_SCORE_THRESHOLD, (
                f"CLIP score {clip_score:.4f} below threshold {CLIP_SCORE_THRESHOLD} "
                f"for prompt '{prompt}'"
            )

        _save_image_and_write_summary(case.id, prompt, content, clip_score)
```
**EN:** This test method exercises diffusion generation and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 diffusion generation 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 153-156: Expose unittest entrypoint
```python
if __name__ == "__main__":
    import sys

    sys.exit(pytest.main([__file__, "-v"]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.multimodal_gen.test.server.test_server_common`, `sglang.multimodal_gen.test.server.test_server_utils`, `sglang.multimodal_gen.test.server.testcase_configs`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `PIL`, `io`, `logging`, `os`, `pytest`, `sys`, `torch`, `transformers`
- Notable symbols / 关键符号: `register_amd_ci`
