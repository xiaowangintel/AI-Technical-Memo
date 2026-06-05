# post_processing.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/api/post_processing.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang diffusion supports optional post-processing steps that run after generation to improve temporal smoothness (frame interpolation) or spatial resolution (upscaling). These steps are independent of the diffusion model and can be combined in a single run. / 该文档围绕 Post-Processing 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang diffusion supports optional post-processing steps that run after generation to improve temporal smoothness (frame interpolation) or spatial resolution (upscaling). These steps are independent of the diffusion model and can be combined in a single run.
**CN:** 本节围绕 Overview 展开，概述了 frame, run, runs, These 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Frame Interpolation (video only)
**EN:** Frame interpolation synthesizes new frames between each pair of consecutive generated frames, producing smoother motion without re-running the diffusion model.
**CN:** 本节围绕 Frame Interpolation (video only) 展开，概述了 frames, new, exp, frame 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CLI Arguments
**EN:** This section provides a comparison table for CLI Arguments, covering columns such as Argument, Description and examples such as --enable-frame-interpolation, --frame-interpolation-exp, --frame-interpolation-scale, --frame-interpolation-model-path.
**CN:** 本节围绕 CLI Arguments 展开，概述了 RIFE, EXP, PATH,  |
|  等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Models
**EN:** This section provides a comparison table for Supported Models, covering columns such as Weight, HuggingFace Repo, Description and examples such as RIFE 4.22.lite *(default)*.
**CN:** 本节围绕 Supported 模型s 展开，概述了 RIFE, IFNet, IFBlock, AMT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example
**EN:** Generate a 5-frame video and interpolate to 9 frames ((5 − 1) × 2¹ + 1 = 9): ``bash sglang generate \ --model-path Wan-AI/Wan2.2-T2V-A14B-Diffusers \ --prompt "A dog running through a park" \ --num-frames 5 \ --enable-frame-interpolation \ --frame-interpolation-exp 1 \ --save-output ``
**CN:** 本节围绕 Example 展开，概述了 Generate, dog, park, frame 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Upscaling (image and video)
**EN:** Upscaling increases the spatial resolution of generated images or video frames using Real-ESRGAN. The model weights are downloaded automatically on first use and cached for subsequent runs.
**CN:** 本节围绕 Upscaling (image and video) 展开，概述了 Real-ESRGAN, Upscaling, runs, video 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CLI Arguments
**EN:** This section provides a comparison table for CLI Arguments, covering columns such as Argument, Description and examples such as --enable-upscaling, --upscaling-scale, --upscaling-model-path.
**CN:** 本节围绕 CLI Arguments 展开，概述了 Real-ESRGAN, HuggingFace, repo_id:filename, RealESRGAN_x4.pth 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Models
**EN:** This section provides a comparison table for Supported Models, covering columns such as Architecture, Example Weights, Description and examples such as **RRDBNet**, **SRVGGNetCompact**.
**CN:** 本节围绕 Supported 模型s 展开，概述了 Real-ESRGAN, RealESRGAN_x4.pth, SRVGGNetCompact, RealESRGAN_x4plus.pth 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Examples
**EN:** This content focuses on Examples and highlights Generate, prompt, upscale, --prompt.
**CN:** 本节围绕 Examples 展开，概述了 Generate, prompt, upscale, --prompt 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Combining Frame Interpolation and Upscaling
**EN:** Frame interpolation and upscaling can be combined in a single run. Interpolation is applied first (increasing the frame count), then upscaling is applied to every frame (increasing the spatial resolution).
**CN:** 本节围绕 Combining Frame Interpolation and Upscaling 展开，概述了 frame, frames, applied, generate 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** frame / **CN:** frame
- **EN:** Real-ESRGAN / **CN:** Real-ESRGAN
- **EN:** RIFE / **CN:** RIFE
- **EN:** frames / **CN:** frames
- **EN:** RealESRGAN_x4.pth / **CN:** RealESRGAN_x4.pth
- **EN:** Description / **CN:** Description
- **EN:** interpolation / **CN:** interpolation
- **EN:** model / **CN:** 模型

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
