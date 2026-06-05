# server_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/apps/ComfyUI_SGLDiffusion/core/server_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the ComfyUI integration layer. It centers on `SGLDiffusionServerAPI`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: SGLang Diffusion Server API client. Provides a low-level interface for interacting with SGLang Diffusion HTTP server. / 该文件属于ComfyUI 集成层。它围绕 `SGLDiffusionServerAPI` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module setup and imports / 模块初始化与导入
```python
"""
SGLang Diffusion Server API client.
Provides a low-level interface for interacting with SGLang Diffusion HTTP server.
"""

import base64
import io
import os
import time
from typing import Any, Dict, Optional

import requests
from PIL import Image
```
**EN:** This block establishes the module context and imports `base64`, `io`, `os`, `time`, `typing`, and `requests`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `base64`、`io`、`os`、`time`、`typing` 和 `requests`。这些依赖为后续实现提供所需符号。

### Lines 16-18: `SGLDiffusionServerAPI` class overview / `SGLDiffusionServerAPI` 类概览
```python
class SGLDiffusionServerAPI:
    """Client for SGLang Diffusion HTTP server API."""
```
**EN:** This block defines class `SGLDiffusionServerAPI`. Client for SGLang Diffusion HTTP server API.
**CN:** 该代码块定义了类 `SGLDiffusionServerAPI`。 它用于封装 sgldiffusion server api 相关行为。

### Lines 19-39: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, base_url: str, api_key: str = "sk-proj-1234567890"):
        """
        Initialize the API client.

        Args:
            base_url: Base URL of the SGLang Diffusion server (e.g., "http://localhost:30010/v1")
            api_key: API key for authentication (default: "sk-proj-1234567890")
        """
        # Ensure base_url doesn't end with /v1 if it's already there
        if base_url.endswith("/v1"):
            self.base_url = base_url
        elif base_url.endswith("/v1/"):
            self.base_url = base_url.rstrip("/")
        else:
            self.base_url = f"{base_url.rstrip('/')}/v1"

        self.api_key = api_key
        self.headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {api_key}",
        }
```
**EN:** This block defines method `__init__` on `SGLDiffusionServerAPI`. Initialize the API client. Args: base_url: Base URL of the SGLang Diffusion server (e.g., "http://localhost:30010/v1") api_key: API key for authentication (default: "sk-proj-1234567890") Key calls include `base_url.endswith`, and `base_url.rstrip`. The implementation branches on conditions. Parameters such as `base_url`, and `api_key` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerAPI` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `base_url.endswith` 和 `base_url.rstrip`。 实现中包含条件分支。 本段逻辑主要由 `base_url` 和 `api_key` 等参数驱动。

### Lines 41-61: `get_model_info` implementation / `get_model_info` 实现
```python
    def get_model_info(self) -> Dict[str, Any]:
        """
        Get information about the model served by this server.

        Returns:
            Dictionary containing model information including:
            - model_path: Path to the model
            - task_type: Type of task (e.g., "T2V", "I2I")
            - pipeline_name: Name of the pipeline
            - num_gpus: Number of GPUs
            - dit_precision: DiT model precision
            - vae_precision: VAE model precision
        """
        try:
            # Remove /v1 from base_url for /models endpoint
            models_url = self.base_url.removesuffix("/v1") + "/models"
            response = requests.get(models_url, headers=self.headers, timeout=30)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            raise RuntimeError(f"Failed to get model info: {str(e)}")
```
**EN:** This block defines method `get_model_info` on `SGLDiffusionServerAPI`. Get information about the model served by this server. Returns: Dictionary containing model information including: - model_path: Path to the model - task_type: Type of task (e.g., "T2V", "I2I") - pipeline_name: Name of the pipeline - num_gpus: Number of GPUs - dit_precision: DiT model precision - vae_precision: VAE model precision Key calls include `requests.get`, `response.raise_for_status`, `response.json`, `self.base_url.removesuffix`, and `RuntimeError`. The implementation handles exceptional paths.
**CN:** 该代码块定义了 `SGLDiffusionServerAPI` 的方法 `get_model_info`。 它用于获取model info。 关键调用包括 `requests.get`、`response.raise_for_status`、`response.json`、`self.base_url.removesuffix` 和 `RuntimeError`。 实现中处理异常路径。

### Lines 63-203: `generate_image` implementation / `generate_image` 实现
```python
    def generate_image(
        self,
        prompt: str,
        image_path: Optional[str] = None,
        mask_path: Optional[str] = None,
        size: Optional[str] = None,
        width: Optional[int] = None,
        height: Optional[int] = None,
        n: int = 1,
        negative_prompt: Optional[str] = None,
        guidance_scale: Optional[float] = None,
        num_inference_steps: Optional[int] = None,
        seed: Optional[int] = None,
        enable_teacache: bool = False,
        response_format: str = "b64_json",
        quality: Optional[str] = "auto",
        style: Optional[str] = "vivid",
        background: Optional[str] = "auto",
        output_format: Optional[str] = None,
        generator_device: Optional[str] = "cuda",
    ) -> Dict[str, Any]:
        """
        Generate or edit an image using SGLang Diffusion API.
        If image_path is provided, calls the edit endpoint; otherwise calls the generation endpoint.

        Args:
            prompt: Text prompt for image generation/editing
            image_path: Optional path to input image file for editing. If provided, uses edit API.
            mask_path: Optional path to mask image file (only used when image_path is provided)
            size: Image size in format "WIDTHxHEIGHT" (e.g., "1024x1024")
            width: Image width (used if size is not provided)
            height: Image height (used if size is not provided)
            n: Number of images to generate (1-10)
            negative_prompt: Negative prompt to avoid certain elements
            guidance_scale: Classifier-free guidance scale
            num_inference_steps: Number of denoising steps
            seed: Random seed for reproducible generation
            enable_teacache: Enable TEA cache acceleration
            response_format: Response format ("b64_json" or "url")
            quality: Image quality ("auto", "standard", "hd") - only for generation
            style: Image style ("vivid" or "natural") - only for generation
            background: Background type ("auto", "transparent", "opaque")
            output_format: Output format ("png", "jpeg", "webp")
            generator_device: Device for random generator ("cuda" or "cpu")

        Returns:
            Dictionary containing the API response with generated/edited image data
        """
        if not prompt:
            raise ValueError("Prompt cannot be empty")

        # Determine size
        if size is None:
            if width is not None and height is not None:
                size = f"{width}x{height}"
            else:
                size = "1024x1024"

        # Build common parameters
        common_params = self._build_image_common_params(
            prompt=prompt,
            size=size,
            n=n,
            response_format=response_format,
            negative_prompt=negative_prompt,
            guidance_scale=guidance_scale,
            num_inference_steps=num_inference_steps,
            seed=seed,
            enable_teacache=enable_teacache,
            background=background,
            output_format=output_format,
            generator_device=generator_device,
        )

        # If image_path is provided, use edit endpoint
        if image_path:
            if not os.path.exists(image_path):
                raise FileNotFoundError(f"Image file not found: {image_path}")

            # Prepare multipart form data for edit
            files: Dict[str, Any] = {}
            data = common_params.copy()

            # Add image file
            files["image"] = (
                os.path.basename(image_path),
                open(image_path, "rb"),
                self._get_content_type(image_path),
            )

            # Add mask file if provided
            if mask_path:
                if not os.path.exists(mask_path):
                    raise FileNotFoundError(f"Mask file not found: {mask_path}")
                files["mask"] = (
                    os.path.basename(mask_path),
                    open(mask_path, "rb"),
                    self._get_content_type(mask_path),
                )

            # Prepare headers for multipart form data
            headers = {
                "Authorization": f"Bearer {self.api_key}",
            }

            try:
                response = requests.post(
                    f"{self.base_url}/images/edits",
                    files=files,
                    data=data,
                    headers=headers,
                    timeout=300,  # 5 minutes timeout for generation
                )
                response.raise_for_status()
                return response.json()
            except requests.exceptions.RequestException as e:
                raise RuntimeError(f"Failed to edit image: {str(e)}")
            finally:
                # Close file handles
                for file_tuple in files.values():
                    if isinstance(file_tuple, tuple) and len(file_tuple) > 1:
                        file_tuple[1].close()
        else:
            # Use generation endpoint - add generation-specific parameters
            payload = common_params.copy()
            if quality:
                payload["quality"] = quality
            if style:
                payload["style"] = style

            try:
                response = requests.post(
                    f"{self.base_url}/images/generations",
                    json=payload,
                    headers=self.headers,
                    timeout=300,  # 5 minutes timeout for generation
                )
                response.raise_for_status()
                return response.json()
            except requests.exceptions.RequestException as e:
                raise RuntimeError(f"Failed to generate image: {str(e)}")
```
**EN:** This block defines method `generate_image` on `SGLDiffusionServerAPI`. Generate or edit an image using SGLang Diffusion API. If image_path is provided, calls the edit endpoint; otherwise calls the generation endpoint. Key calls include `self._build_image_common_params`, `ValueError`, `common_params.copy`, `os.path.exists`, and `FileNotFoundError`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `prompt`, `image_path`, `mask_path`, `size`, and `width` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerAPI` 的方法 `generate_image`。 它用于生成image。 关键调用包括 `self._build_image_common_params`、`ValueError`、`common_params.copy`、`os.path.exists` 和 `FileNotFoundError`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `prompt`、`image_path`、`mask_path`、`size` 和 `width` 等参数驱动。

### Lines 205-349: `generate_video` implementation / `generate_video` 实现
```python
    def generate_video(
        self,
        prompt: str,
        size: Optional[str] = None,
        width: Optional[int] = None,
        height: Optional[int] = None,
        seconds: Optional[int] = 4,
        fps: Optional[int] = None,
        num_frames: Optional[int] = None,
        negative_prompt: Optional[str] = None,
        guidance_scale: Optional[float] = None,
        num_inference_steps: Optional[int] = None,
        seed: Optional[int] = None,
        enable_teacache: bool = False,
        generator_device: Optional[str] = "cuda",
        input_reference: Optional[str] = None,
        output_path: Optional[str] = None,
    ) -> Dict[str, Any]:
        """
        Generate a video using SGLang Diffusion API and wait for completion.

        Args:
            prompt: Text prompt for video generation
            size: Video size in format "WIDTHxHEIGHT" (e.g., "1280x720")
            width: Video width (used if size is not provided)
            height: Video height (used if size is not provided)
            seconds: Duration of the video in seconds
            fps: Frames per second
            num_frames: Number of frames (overrides seconds * fps if provided)
            negative_prompt: Negative prompt to avoid certain elements
            guidance_scale: Classifier-free guidance scale
            num_inference_steps: Number of denoising steps
            seed: Random seed for reproducible generation
            enable_teacache: Enable TEA cache acceleration
            generator_device: Device for random generator ("cuda" or "cpu")
            input_reference: Path to input reference image for image-to-video

        Returns:
            Dictionary containing completed video job information with file_path
        """
        if not prompt:
            raise ValueError("Prompt cannot be empty")

        # Determine size
        if size is None:
            if width is not None and height is not None:
                size = f"{width}x{height}"
            else:
                size = "720x1280"

        # Prepare request payload
        payload: Dict[str, Any] = {
            "prompt": prompt,
            "size": size,
        }

        # Add optional parameters
        if seconds is not None:
            payload["seconds"] = seconds
        if fps is not None:
            payload["fps"] = fps
        if num_frames is not None:
            payload["num_frames"] = num_frames
        if negative_prompt:
            payload["negative_prompt"] = negative_prompt
        if guidance_scale is not None:
            payload["guidance_scale"] = guidance_scale
        if num_inference_steps is not None:
            payload["num_inference_steps"] = num_inference_steps
        if seed is not None and seed >= 0:
            payload["seed"] = seed
        if enable_teacache:
            payload["enable_teacache"] = True
        if generator_device:
            payload["generator_device"] = generator_device
        if input_reference:
            payload["input_reference"] = input_reference
        if output_path:
            payload["output_path"] = output_path

        try:
            # Create video generation job
            response = requests.post(
                f"{self.base_url}/videos",
                json=payload,
                headers=self.headers,
                timeout=30,
            )
            response.raise_for_status()
            video_job = response.json()
            video_id = video_job.get("id")

            # Wait for completion with fixed polling
            poll_interval = 5  # 5 seconds
            max_wait_time = 3600  # 1 hour
            max_consecutive_errors = 5
            consecutive_errors = 0
            start_time = time.time()

            while time.time() - start_time < max_wait_time:
                try:
                    status_response = requests.get(
                        f"{self.base_url}/videos/{video_id}",
                        headers=self.headers,
                        timeout=30,
                    )
                    status_response.raise_for_status()
                    status = status_response.json()

                    # Reset error counter on successful request
                    consecutive_errors = 0

                    if status.get("status") == "completed":
                        return status
                    elif status.get("status") == "failed":
                        error = status.get("error", {})
                        error_msg = (
                            error.get("message", "Unknown error")
                            if error
                            else "Unknown error"
                        )
                        raise RuntimeError(f"Video generation failed: {error_msg}")
                except requests.exceptions.ConnectionError as e:
                    # Connection errors - likely server is down
                    consecutive_errors += 1
                    if consecutive_errors >= max_consecutive_errors:
                        raise RuntimeError(
                            f"Lost connection to server after {consecutive_errors} consecutive errors. "
                            f"Server may be unavailable: {str(e)}"
                        )
                except requests.exceptions.RequestException as e:
                    # Other network errors - continue polling but track errors
                    consecutive_errors += 1
                    if consecutive_errors >= max_consecutive_errors:
                        raise RuntimeError(
                            f"Network error after {consecutive_errors} consecutive failures: {str(e)}"
                        )

                time.sleep(poll_interval)

            raise TimeoutError(
                f"Video generation timed out after {max_wait_time} seconds"
            )
        except requests.exceptions.RequestException as e:
            raise RuntimeError(f"Failed to generate video: {str(e)}")
```
**EN:** This block defines method `generate_video` on `SGLDiffusionServerAPI`. Generate a video using SGLang Diffusion API and wait for completion. Args: prompt: Text prompt for video generation size: Video size in format "WIDTHxHEIGHT" (e.g., "1280x720") width: Video width (used if size is not provided) height: Video height (used if size is not provided) seconds: Duration of the video in seconds fps: Frames per second num_frames: Number of frames (overrides seconds * fps if provided) negative_prompt: Negative prompt to avoid certain elements guidance_scale: Classifier-free guidance scale num_inference_steps: Number of denoising steps seed: Random seed for reproducible generation enable_teacache: Enable TEA cache acceleration generator_device: Device for random generator ("cuda" or "cpu") input_reference: Path to input reference image for image-to-video Returns: Dictionary containing completed video job information with file_path Key calls include `ValueError`, `requests.post`, `response.raise_for_status`, `response.json`, and `video_job.get`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `prompt`, `size`, `width`, `height`, and `seconds` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerAPI` 的方法 `generate_video`。 它用于生成video。 关键调用包括 `ValueError`、`requests.post`、`response.raise_for_status`、`response.json` 和 `video_job.get`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `prompt`、`size`、`width`、`height` 和 `seconds` 等参数驱动。

### Lines 351-397: `_build_image_common_params` implementation / `_build_image_common_params` 实现
```python
    def _build_image_common_params(
        self,
        prompt: str,
        size: str,
        n: int,
        response_format: str,
        negative_prompt: Optional[str] = None,
        guidance_scale: Optional[float] = None,
        num_inference_steps: Optional[int] = None,
        seed: Optional[int] = None,
        enable_teacache: bool = False,
        background: Optional[str] = None,
        output_format: Optional[str] = None,
        generator_device: Optional[str] = None,
    ) -> Dict[str, Any]:
        """
        Build common parameters for both image generation and editing.

        Returns:
            Dictionary containing common parameters
        """
        params: Dict[str, Any] = {
            "prompt": prompt,
            "size": size,
            "n": max(1, min(n, 10)),
            "response_format": response_format,
        }

        # Add optional parameters
        if negative_prompt:
            params["negative_prompt"] = negative_prompt
        if guidance_scale is not None:
            params["guidance_scale"] = guidance_scale
        if num_inference_steps is not None:
            params["num_inference_steps"] = num_inference_steps
        if seed is not None and seed >= 0:
            params["seed"] = seed
        if enable_teacache:
            params["enable_teacache"] = True
        if background:
            params["background"] = background
        if output_format:
            params["output_format"] = output_format
        if generator_device:
            params["generator_device"] = generator_device

        return params
```
**EN:** This block defines method `_build_image_common_params` on `SGLDiffusionServerAPI`. Build common parameters for both image generation and editing. Returns: Dictionary containing common parameters Key calls include `max`, and `min`. The implementation branches on conditions. Parameters such as `prompt`, `size`, `n`, `response_format`, and `negative_prompt` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerAPI` 的方法 `_build_image_common_params`。 它用于构建image common params。 关键调用包括 `max` 和 `min`。 实现中包含条件分支。 本段逻辑主要由 `prompt`、`size`、`n`、`response_format` 和 `negative_prompt` 等参数驱动。

### Lines 399-408: `_get_content_type` implementation / `_get_content_type` 实现
```python
    def _get_content_type(self, file_path: str) -> str:
        """Get content type based on file extension."""
        ext = os.path.splitext(file_path)[1].lower()
        content_types = {
            ".png": "image/png",
            ".jpg": "image/jpeg",
            ".jpeg": "image/jpeg",
            ".webp": "image/webp",
        }
        return content_types.get(ext, "image/png")
```
**EN:** This block defines method `_get_content_type` on `SGLDiffusionServerAPI`. Get content type based on file extension. Key calls include `os.path.splitext.lower`, `content_types.get`, and `os.path.splitext`. Parameters such as `file_path` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerAPI` 的方法 `_get_content_type`。 它用于获取content type。 关键调用包括 `os.path.splitext.lower`、`content_types.get` 和 `os.path.splitext`。 本段逻辑主要由 `file_path` 等参数驱动。

### Lines 410-440: `decode_image_from_response` implementation / `decode_image_from_response` 实现
```python
    def decode_image_from_response(
        self, response_data: Dict[str, Any], index: int = 0
    ) -> Image.Image:
        """
        Decode base64 image from API response.

        Args:
            response_data: API response dictionary
            index: Index of the image in the response (default: 0)

        Returns:
            PIL Image object
        """
        if "data" not in response_data or not response_data["data"]:
            raise ValueError("No image data in response")

        if index >= len(response_data["data"]):
            raise IndexError(f"Image index {index} out of range")

        image_data = response_data["data"][index]
        if "b64_json" not in image_data or not image_data["b64_json"]:
            raise ValueError("No base64 image data found")

        image_bytes = base64.b64decode(image_data["b64_json"])
        image = Image.open(io.BytesIO(image_bytes))

        # Convert to RGB if needed
        if image.mode != "RGB":
            image = image.convert("RGB")

        return image
```
**EN:** This block defines method `decode_image_from_response` on `SGLDiffusionServerAPI`. Decode base64 image from API response. Args: response_data: API response dictionary index: Index of the image in the response (default: 0) Returns: PIL Image object Key calls include `base64.b64decode`, `Image.open`, `ValueError`, `len`, and `IndexError`. The implementation branches on conditions. Parameters such as `response_data`, and `index` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerAPI` 的方法 `decode_image_from_response`。 它用于解码image from response。 关键调用包括 `base64.b64decode`、`Image.open`、`ValueError`、`len` 和 `IndexError`。 实现中包含条件分支。 本段逻辑主要由 `response_data` 和 `index` 等参数驱动。

### Lines 442-487: `set_lora` implementation / `set_lora` 实现
```python
    def set_lora(
        self,
        lora_nickname: str,
        lora_path: Optional[str] = None,
        target: str = "all",
    ) -> Dict[str, Any]:
        """
        Set a LoRA adapter for the specified transformer(s).

        Args:
            lora_nickname: The nickname of the adapter (required).
            lora_path: Path to the LoRA adapter (local path or HF repo id).
                      Required for the first load; optional if re-activating a cached nickname.
            target: Which transformer(s) to apply the LoRA to. One of:
                - "all": Apply to all transformers (default)
                - "transformer": Apply only to the primary transformer (high noise for Wan2.2)
                - "transformer_2": Apply only to transformer_2 (low noise for Wan2.2)
                - "critic": Apply only to the critic model

        Returns:
            Dictionary containing the API response with status and message
        """
        if not lora_nickname:
            raise ValueError("lora_nickname cannot be empty")

        # Prepare request payload
        payload: Dict[str, Any] = {
            "lora_nickname": lora_nickname,
            "target": target,
        }

        # Add optional lora_path if provided
        if lora_path:
            payload["lora_path"] = lora_path

        try:
            response = requests.post(
                f"{self.base_url}/set_lora",
                json=payload,
                headers=self.headers,
                timeout=30,
            )
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            raise RuntimeError(f"Failed to set LoRA adapter: {str(e)}")
```
**EN:** This block defines method `set_lora` on `SGLDiffusionServerAPI`. Set a LoRA adapter for the specified transformer(s). Args: lora_nickname: The nickname of the adapter (required). Key calls include `ValueError`, `requests.post`, `response.raise_for_status`, `response.json`, and `RuntimeError`. The implementation branches on conditions, handles exceptional paths. Parameters such as `lora_nickname`, `lora_path`, and `target` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerAPI` 的方法 `set_lora`。 它用于设置lora。 关键调用包括 `ValueError`、`requests.post`、`response.raise_for_status`、`response.json` 和 `RuntimeError`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `lora_nickname`、`lora_path` 和 `target` 等参数驱动。

### Lines 489-517: `unset_lora` implementation / `unset_lora` 实现
```python
    def unset_lora(
        self,
        target: str = "all",
    ) -> Dict[str, Any]:
        """
        Unset (unmerge) LoRA weights from the base model.

        Args:
            target: same as set_lora

        Returns:
            Dictionary containing the API response with status and message
        """
        # Prepare request payload
        payload: Dict[str, Any] = {
            "target": target,
        }

        try:
            response = requests.post(
                f"{self.base_url}/unmerge_lora_weights",
                json=payload,
                headers=self.headers,
                timeout=30,
            )
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            raise RuntimeError(f"Failed to unset LoRA adapter: {str(e)}")
```
**EN:** This block defines method `unset_lora` on `SGLDiffusionServerAPI`. Unset (unmerge) LoRA weights from the base model. Args: target: same as set_lora Returns: Dictionary containing the API response with status and message Key calls include `requests.post`, `response.raise_for_status`, `response.json`, `RuntimeError`, and `str`. The implementation handles exceptional paths. Parameters such as `target` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerAPI` 的方法 `unset_lora`。 它用于处理 unset lora 相关逻辑。 关键调用包括 `requests.post`、`response.raise_for_status`、`response.json`、`RuntimeError` 和 `str`。 实现中处理异常路径。 本段逻辑主要由 `target` 等参数驱动。

### Lines 520-539: supporting statements / 辅助语句
```python
if __name__ == "__main__":
    api = SGLDiffusionServerAPI(
        base_url="http://localhost:30010/v1", api_key="sk-proj-1234567890"
    )
    model_info = api.get_model_info()
    print(api.get_model_info())
    if model_info.get("task_type") == "T2V" or model_info.get("task_type") == "I2V":
        print(
            api.generate_video(
                prompt="A calico cat playing a piano on stage",
                num_inference_steps=1,
                size="480x480",
            )
        )
    else:
        print(
            api.generate_image(
                prompt="A calico cat playing a piano on stage", size="1024x1024"
            )
        )
```
**EN:** This block gathers supporting statements at module scope. The code collaborates with `SGLDiffusionServerAPI`, `api.get_model_info`, `print`, and `model_info.get`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 代码会与 `SGLDiffusionServerAPI`、`api.get_model_info`、`print` 和 `model_info.get` 协同工作。

## Key Concepts / 关键概念
- `SGLDiffusionServerAPI`: Client for SGLang Diffusion HTTP server API. / 核心类，用于封装 sgldiffusion server api 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `base64`, `io`, `os`, `time`, `typing`
- **Third-party / 第三方依赖**: `requests`, `PIL`

- **Total lines / 总行数**: 539
