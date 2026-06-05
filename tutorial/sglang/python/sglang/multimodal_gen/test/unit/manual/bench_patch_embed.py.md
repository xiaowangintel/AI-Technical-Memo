# bench_patch_embed.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/manual/bench_patch_embed.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates bench patch embed with focused assertions and fixtures. Key symbols include `PatchEmbed3D`, `PatchEmbed`, `_copy_weights`. / 该测试模块通过有针对性的断言与夹具，验证 bench patch embed 的实现。 关键符号包括 `PatchEmbed3D`, `PatchEmbed`, `_copy_weights`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports and module setup / 导入与模块初始化
```python
"""
Benchmark: Conv3d vs reshape + F.linear PatchEmbed.

Matches the real e2e pipeline conditions:
  - Conv3d weights are FP32 (no dtype passed to PatchEmbed.__init__)
  - Input latents are BF16 (cast by denoising loop)
  - torch.autocast(dtype=bf16) wraps the forward pass
  - .flatten(2).transpose(1, 2) follows PatchEmbed (wanvideo.py:1008)

Uses CUDA events for accurate GPU timing. Each case runs warmup iterations
followed by timed iterations, reports median latency and speedup.

Usage:
    python bench_patch_embed.py
# ...

import torch
import torch.nn as nn
import torch.nn.functional as F
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 25-49: Class `PatchEmbed3D` / 类 `PatchEmbed3D`
```python
class PatchEmbed3D(nn.Module):
    """Conv3d-based PatchEmbed (upstream/main)."""

    def __init__(self, patch_size, in_chans, embed_dim, flatten=True, bias=True):
        super().__init__()
        if isinstance(patch_size, list | tuple):
            if len(patch_size) == 1:
                patch_size = (patch_size[0], patch_size[0])
        else:
            patch_size = (patch_size, patch_size)
        self.patch_size = patch_size
        self.flatten = flatten
        self.proj = nn.Conv3d(
            in_chans,
# ...
        x = self.proj(x)
        if self.flatten:
            x = x.flatten(2).transpose(1, 2)
        return x
```
**EN:** This class models `PatchEmbed3D` as a specialization of `nn.Module`. Conv3d-based PatchEmbed (upstream/main). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `PatchEmbed3D`，并继承/扩展 `nn.Module`。 文档字符串指出：Conv3d-based PatchEmbed (upstream/main). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 52-87: Class `PatchEmbed` / 类 `PatchEmbed`
```python
class PatchEmbed(nn.Module):
    """Reshape + F.linear PatchEmbed (opt_krea)."""

    def __init__(self, patch_size, in_chans, embed_dim, flatten=True, bias=True):
        super().__init__()
        if isinstance(patch_size, list | tuple):
            if len(patch_size) == 1:
                patch_size = (1, patch_size[0], patch_size[0])
            elif len(patch_size) == 2:
                patch_size = (1, patch_size[0], patch_size[1])
        else:
            patch_size = (1, patch_size, patch_size)
        self.patch_size = patch_size
        self.flatten = flatten
# ...
        x = F.linear(x, w, self.proj.bias)
        if not self.flatten:
            x = x.reshape(B, T_, H_, W_, -1).permute(0, 4, 1, 2, 3).contiguous()
        return x
```
**EN:** This class models `PatchEmbed` as a specialization of `nn.Module`. Reshape + F.linear PatchEmbed (opt_krea). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `PatchEmbed`，并继承/扩展 `nn.Module`。 文档字符串指出：Reshape + F.linear PatchEmbed (opt_krea). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 90-93: Function `_copy_weights` / 函数 `_copy_weights`
```python
def _copy_weights(src, dst):
    dst.proj.weight.data.copy_(src.proj.weight.data)
    if src.proj.bias is not None:
        dst.proj.bias.data.copy_(src.proj.bias.data)
```
**EN:** This function drives `_copy_weights` with inputs such as `src`, `dst`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_copy_weights`，主要处理 `src`, `dst` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 96-111: Function `bench_one` / 函数 `bench_one`
```python
def bench_one(fn, warmup, iters):
    """Returns list of per-iteration latencies in ms using CUDA events."""
    for _ in range(warmup):
        fn()
    torch.cuda.synchronize()

    times = []
    for _ in range(iters):
        start = torch.cuda.Event(enable_timing=True)
        end = torch.cuda.Event(enable_timing=True)
        start.record()
        fn()
        end.record()
        torch.cuda.synchronize()
        times.append(start.elapsed_time(end))
    return times
```
**EN:** This function drives `bench_one` with inputs such as `fn`, `warmup`, `iters`. Returns list of per-iteration latencies in ms using CUDA events.
**CN:** 这个函数负责 `bench_one`，主要处理 `fn`, `warmup`, `iters` 等输入。 文档字符串说明：Returns list of per-iteration latencies in ms using CUDA events.

### Lines 112-128: Top-level configuration / 顶层配置
```python


# Real latent shapes: T = (num_frames-1)//4+1, H = height//8, W = width//8
# (name, patch_size, in_chans, embed_dim, flatten, B, T, H, W)
BENCH_CASES = [
    # Wan2.1-I2V-14B: 480x832
    ("Wan-21f-480x832", (1, 2, 2), 16, 5120, False, 1, 6, 60, 104),  # 21 frames
    ("Wan-41f-480x832", (1, 2, 2), 16, 5120, False, 1, 11, 60, 104),  # 41 frames
    ("Wan-81f-480x832", (1, 2, 2), 16, 5120, False, 1, 21, 60, 104),  # 81 frames
    ("Wan-101f-480x832", (1, 2, 2), 16, 5120, False, 1, 26, 60, 104),  # 101 frames
    # Wan2.1-I2V-14B: 720x1280
    ("Wan-21f-720x1280", (1, 2, 2), 16, 5120, False, 1, 6, 90, 160),  # 21 frames 720p
    ("Wan-41f-720x1280", (1, 2, 2), 16, 5120, False, 1, 11, 90, 160),  # 41 frames 720p
    # HunyuanVideo
    ("HunYuan-21f-480x832", (1, 2, 2), 16, 3072, True, 1, 6, 60, 104),
    ("HunYuan-41f-480x832", (1, 2, 2), 16, 3072, True, 1, 11, 60, 104),
]
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 131-203: Function `main` / 函数 `main`
```python
def main():
    parser = argparse.ArgumentParser(
        description="Benchmark PatchEmbed: Conv3d vs F.linear"
    )
    parser.add_argument("--warmup", type=int, default=10)
    parser.add_argument("--iters", type=int, default=50)
    args = parser.parse_args()

    device = "cuda"

    # ── Real pipeline conditions ──────────────────────────────────────────
    # 1. Weights are FP32 (PatchEmbed.__init__ has no dtype arg in real code)
    # 2. Input is BF16 (latents.to(target_dtype) in denoising loop)
    # 3. torch.autocast(dtype=bf16) wraps the denoising loop
# ...

        print(f"{name:<25} {med_conv:>10.3f} {med_lin:>12.3f} {speedup:>7.2f}x")

    print()
```
**EN:** This function drives `main`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `main`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 204-207: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Automated verification / 自动化验证
- Video generation flow / 视频生成流程
- Symbol `PatchEmbed3D` anchors the module API / 符号 `PatchEmbed3D` 构成该模块的核心 API

## Dependencies / 依赖关系
- **External / 外部**: `torch`, `torch.nn`, `torch.nn.functional`
- **Stdlib / 标准库**: `argparse`
