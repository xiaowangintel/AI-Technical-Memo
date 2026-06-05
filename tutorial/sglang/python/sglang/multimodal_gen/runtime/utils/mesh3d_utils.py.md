# mesh3d_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/utils/mesh3d_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for mesh3d utils in the multimodal generation stack. Key symbols include `transform_pos`, `get_mv_matrix`, `get_orthographic_projection_matrix`. / 该模块包含多模态生成体系中与 mesh3d utils 相关的运行时支持代码。 关键符号包括 `transform_pos`, `get_mv_matrix`, `get_orthographic_projection_matrix`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and module setup / 导入与模块初始化
```python
"""Adapted from Hunyuan3D-2: https://github.com/Tencent/Hunyuan3D-2"""

from __future__ import annotations

import math
from typing import Any, List, Optional, Tuple, Union

import cv2
import numpy as np
import torch
import torch.nn.functional as F
import trimesh
from einops import rearrange, repeat
from PIL import Image
# ...
logger = init_logger(__name__)

# Import C++ mesh processor extension
from sglang.multimodal_gen.csrc.render.mesh_processor import meshVerticeInpaint
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 24-40: Function `transform_pos` / 函数 `transform_pos`
```python
def transform_pos(
    mtx: Union[np.ndarray, torch.Tensor],
    pos: torch.Tensor,
    keepdim: bool = False,
) -> torch.Tensor:
    """Transform positions by a matrix."""
    t_mtx = torch.from_numpy(mtx).to(pos.device) if isinstance(mtx, np.ndarray) else mtx

    if pos.shape[-1] == 3:
        posw = torch.cat([pos, torch.ones([pos.shape[0], 1]).to(pos.device)], axis=1)
    else:
        posw = pos

    if keepdim:
        return torch.matmul(posw, t_mtx.t())[...]
    else:
        return torch.matmul(posw, t_mtx.t())[None, ...]
```
**EN:** This function drives `transform_pos` with inputs such as `mtx`, `pos`, `keepdim`. Transform positions by a matrix.
**CN:** 这个函数负责 `transform_pos`，主要处理 `mtx`, `pos`, `keepdim` 等输入。 文档字符串说明：Transform positions by a matrix.

### Lines 43-87: Function `get_mv_matrix` / 函数 `get_mv_matrix`
```python
def get_mv_matrix(
    elev: float,
    azim: float,
    camera_distance: float,
    center: Optional[np.ndarray] = None,
) -> np.ndarray:
    """Compute model-view matrix from camera parameters."""
    elev = -elev
    azim += 90

    elev_rad = math.radians(elev)
    azim_rad = math.radians(azim)

    camera_position = np.array(
# ...
    w2c[:3, 3:] = -np.matmul(np.transpose(c2w[:3, :3], (1, 0)), c2w[:3, 3:])
    w2c[3, 3] = 1.0

    return w2c.astype(np.float32)
```
**EN:** This function drives `get_mv_matrix` with inputs such as `elev`, `azim`, `camera_distance`, `center`. Compute model-view matrix from camera parameters.
**CN:** 这个函数负责 `get_mv_matrix`，主要处理 `elev`, `azim`, `camera_distance`, `center` 等输入。 文档字符串说明：Compute model-view matrix from camera parameters.

### Lines 90-106: Function `get_orthographic_projection_matrix` / 函数 `get_orthographic_projection_matrix`
```python
def get_orthographic_projection_matrix(
    left: float = -1,
    right: float = 1,
    bottom: float = -1,
    top: float = 1,
    near: float = 0,
    far: float = 2,
) -> np.ndarray:
    """Compute orthographic projection matrix."""
    ortho_matrix = np.eye(4, dtype=np.float32)
    ortho_matrix[0, 0] = 2 / (right - left)
    ortho_matrix[1, 1] = 2 / (top - bottom)
    ortho_matrix[2, 2] = -2 / (far - near)
    ortho_matrix[0, 3] = -(right + left) / (right - left)
    ortho_matrix[1, 3] = -(top + bottom) / (top - bottom)
    ortho_matrix[2, 3] = -(far + near) / (far - near)
    return ortho_matrix
```
**EN:** This function drives `get_orthographic_projection_matrix` with inputs such as `left`, `right`, `bottom`, `top`. Compute orthographic projection matrix.
**CN:** 这个函数负责 `get_orthographic_projection_matrix`，主要处理 `left`, `right`, `bottom`, `top` 等输入。 文档字符串说明：Compute orthographic projection matrix.

### Lines 109-124: Function `get_perspective_projection_matrix` / 函数 `get_perspective_projection_matrix`
```python
def get_perspective_projection_matrix(
    fovy: float,
    aspect_wh: float,
    near: float,
    far: float,
) -> np.ndarray:
    """Compute perspective projection matrix."""
    fovy_rad = math.radians(fovy)
    return np.array(
        [
            [1.0 / (math.tan(fovy_rad / 2.0) * aspect_wh), 0, 0, 0],
            [0, 1.0 / math.tan(fovy_rad / 2.0), 0, 0],
            [0, 0, -(far + near) / (far - near), -2.0 * far * near / (far - near)],
            [0, 0, -1, 0],
        ]
    ).astype(np.float32)
```
**EN:** This function drives `get_perspective_projection_matrix` with inputs such as `fovy`, `aspect_wh`, `near`, `far`. Compute perspective projection matrix.
**CN:** 这个函数负责 `get_perspective_projection_matrix`，主要处理 `fovy`, `aspect_wh`, `near`, `far` 等输入。 文档字符串说明：Compute perspective projection matrix.

### Lines 127-142: Function `export_to_trimesh` / 函数 `export_to_trimesh`
```python
def export_to_trimesh(mesh_output: Any) -> Any:
    """Convert mesh output to trimesh format."""
    if isinstance(mesh_output, list):
        outputs = []
        for mesh in mesh_output:
            if mesh is None:
                outputs.append(None)
            else:
                # Reverse face winding
                mesh.mesh_f = mesh.mesh_f[:, ::-1]
                mesh_obj = trimesh.Trimesh(mesh.mesh_v, mesh.mesh_f)
                outputs.append(mesh_obj)
        return outputs
    else:
        mesh_output.mesh_f = mesh_output.mesh_f[:, ::-1]
        return trimesh.Trimesh(mesh_output.mesh_v, mesh_output.mesh_f)
```
**EN:** This function drives `export_to_trimesh` with inputs such as `mesh_output`. Convert mesh output to trimesh format.
**CN:** 这个函数负责 `export_to_trimesh`，主要处理 `mesh_output` 等输入。 文档字符串说明：Convert mesh output to trimesh format.

### Lines 145-172: Function `mesh_uv_wrap` / 函数 `mesh_uv_wrap`
```python
def mesh_uv_wrap(mesh: Any) -> Any:
    """Apply UV unwrapping to mesh. In-place like native Hunyuan3D-2 for same layout."""
    try:
        import xatlas
    except ImportError:
        logger.warning("xatlas not available, skipping UV unwrap")
        return mesh

    if isinstance(mesh, trimesh.Scene):
        mesh = mesh.dump(concatenate=True)

    if len(mesh.faces) > 500000000:
        raise ValueError(
            "The mesh has more than 500,000,000 faces, which is not supported."
# ...
    else:
        mesh.visual.uv = uvs

    return mesh
```
**EN:** This function drives `mesh_uv_wrap` with inputs such as `mesh`. Apply UV unwrapping to mesh.
**CN:** 这个函数负责 `mesh_uv_wrap`，主要处理 `mesh` 等输入。 文档字符串说明：Apply UV unwrapping to mesh.

### Lines 175-180: Function `stride_from_shape` / 函数 `stride_from_shape`
```python
def stride_from_shape(shape: Tuple[int, ...]) -> List[int]:
    """Compute stride from shape for scatter operations."""
    stride = [1]
    for x in reversed(shape[1:]):
        stride.append(stride[-1] * x)
    return list(reversed(stride))
```
**EN:** This function drives `stride_from_shape` with inputs such as `shape`. Compute stride from shape for scatter operations.
**CN:** 这个函数负责 `stride_from_shape`，主要处理 `shape` 等输入。 文档字符串说明：Compute stride from shape for scatter operations.

### Lines 183-211: Function `scatter_add_nd_with_count` / 函数 `scatter_add_nd_with_count`
```python
def scatter_add_nd_with_count(
    input: torch.Tensor,
    count: torch.Tensor,
    indices: torch.Tensor,
    values: torch.Tensor,
    weights: Optional[torch.Tensor] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Scatter add with counting for texture baking."""
    D = indices.shape[-1]
    C = input.shape[-1]
    size = input.shape[:-1]
    stride = stride_from_shape(size)

    assert len(size) == D
# ...
    input.scatter_add_(0, flatten_indices.unsqueeze(1).repeat(1, C), values)
    count.scatter_add_(0, flatten_indices.unsqueeze(1), weights)

    return input.view(*size, C), count.view(*size, 1)
```
**EN:** This function drives `scatter_add_nd_with_count` with inputs such as `input`, `count`, `indices`, `values`. Scatter add with counting for texture baking.
**CN:** 这个函数负责 `scatter_add_nd_with_count`，主要处理 `input`, `count`, `indices`, `values` 等输入。 文档字符串说明：Scatter add with counting for texture baking.

### Lines 214-286: Function `linear_grid_put_2d` / 函数 `linear_grid_put_2d`
```python
def linear_grid_put_2d(
    H: int,
    W: int,
    coords: torch.Tensor,
    values: torch.Tensor,
    return_count: bool = False,
) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
    """Put values into a 2D grid using linear interpolation."""
    C = values.shape[-1]

    indices = coords * torch.tensor(
        [H - 1, W - 1], dtype=torch.float32, device=coords.device
    )
    indices_00 = indices.floor().long()
# ...
    mask = count.squeeze(-1) > 0
    result[mask] = result[mask] / count[mask].repeat(1, C)

    return result
```
**EN:** This function drives `linear_grid_put_2d` with inputs such as `H`, `W`, `coords`, `values`. Put values into a 2D grid using linear interpolation.
**CN:** 这个函数负责 `linear_grid_put_2d`，主要处理 `H`, `W`, `coords`, `values` 等输入。 文档字符串说明：Put values into a 2D grid using linear interpolation.

### Lines 289-894: Class `MeshRender` / 类 `MeshRender`
```python
class MeshRender:
    """Mesh renderer using CUDA rasterization for texture generation."""

    def __init__(
        self,
        camera_distance: float = 1.45,
        camera_type: str = "orth",
        default_resolution: int = 1024,
        texture_size: int = 1024,
        bake_mode: str = "linear",
        device: str = "cuda",
    ):
        """Initialize the mesh renderer."""
        self.device = device
# ...
        return torch.from_numpy(texture_inpainted / 255.0).float().to(self.device)

    # Alias for compatibility
    uv_inpaint = texture_inpaint
```
**EN:** This class models `MeshRender`. Mesh renderer using CUDA rasterization for texture generation. Important methods include `__init__`, `set_default_render_resolution`, `set_default_texture_resolution`, `_rasterize`.
**CN:** 该类实现 `MeshRender`。 文档字符串指出：Mesh renderer using CUDA rasterization for texture generation. 其中较重要的方法包括 `__init__`, `set_default_render_resolution`, `set_default_texture_resolution`, `_rasterize`。

### Lines 897-903: Function `array_to_tensor` / 函数 `array_to_tensor`
```python
def array_to_tensor(np_array):
    """Convert numpy array to normalized tensor."""
    image_pt = torch.tensor(np_array).float()
    image_pt = image_pt / 255 * 2 - 1
    image_pt = rearrange(image_pt, "h w c -> c h w")
    image_pts = repeat(image_pt, "c h w -> b c h w", b=1)
    return image_pts
```
**EN:** This function drives `array_to_tensor` with inputs such as `np_array`. Convert numpy array to normalized tensor.
**CN:** 这个函数负责 `array_to_tensor`，主要处理 `np_array` 等输入。 文档字符串说明：Convert numpy array to normalized tensor.

### Lines 906-940: Function `recenter_image` / 函数 `recenter_image`
```python
def recenter_image(image, border_ratio=0.2):
    """Recenter a PIL image, cropping to non-transparent content with a border."""
    from PIL import Image as PILImage

    if image.mode == "RGB":
        return image
    elif image.mode == "L":
        return image.convert("RGB")
    if image.mode != "RGBA":
        image = image.convert("RGBA")

    alpha_channel = np.array(image)[:, :, 3]
    non_zero_indices = np.argwhere(alpha_channel > 0)
    if non_zero_indices.size == 0:
# ...
    paste_x = (square_size - new_width) // 2 + border_width
    paste_y = (square_size - new_height) // 2 + border_height
    new_image.paste(cropped_image, (paste_x, paste_y))
    return new_image
```
**EN:** This function drives `recenter_image` with inputs such as `image`, `border_ratio`. Recenter a PIL image, cropping to non-transparent content with a border.
**CN:** 这个函数负责 `recenter_image`，主要处理 `image`, `border_ratio` 等输入。 文档字符串说明：Recenter a PIL image, cropping to non-transparent content with a border.

### Lines 943-1031: Class `ImageProcessorV2` / 类 `ImageProcessorV2`
```python
class ImageProcessorV2:
    """Image processor for Hunyuan3D single-view input."""

    # External module path aliases for compatibility with Hunyuan3D configs
    _aliases = [
        "hy3dshape.preprocessors.ImageProcessorV2",
        "hy3dgen.shapegen.preprocessors.ImageProcessorV2",
    ]

    def __init__(self, size=512, border_ratio=None):
        self.size = size
        self.border_ratio = border_ratio

    @staticmethod
# ...
            image, border_ratio=border_ratio, to_tensor=to_tensor
        )
        outputs = {"image": image, "mask": mask}
        return outputs
```
**EN:** This class models `ImageProcessorV2`. Image processor for Hunyuan3D single-view input. Important methods include `__init__`, `recenter`, `load_image`, `__call__`.
**CN:** 该类实现 `ImageProcessorV2`。 文档字符串指出：Image processor for Hunyuan3D single-view input. 其中较重要的方法包括 `__init__`, `recenter`, `load_image`, `__call__`。

### Lines 1034-1070: Class `MVImageProcessorV2` / 类 `MVImageProcessorV2`
```python
class MVImageProcessorV2(ImageProcessorV2):
    """Multi-view image processor for Hunyuan3D."""

    # External module path aliases for compatibility with Hunyuan3D configs
    _aliases = [
        "hy3dshape.preprocessors.MVImageProcessorV2",
    ]

    return_view_idx = True

    def __init__(self, size=512, border_ratio=None):
        super().__init__(size, border_ratio)
        self.view2idx = {"front": 0, "left": 1, "back": 2, "right": 3}

# ...
        image = torch.cat(images, 0).unsqueeze(0)
        mask = torch.cat(masks, 0).unsqueeze(0)
        outputs = {"image": image, "mask": mask, "view_idxs": view_idxs}
        return outputs
```
**EN:** This class models `MVImageProcessorV2` as a specialization of `ImageProcessorV2`. Multi-view image processor for Hunyuan3D. Important methods include `__init__`, `__call__`.
**CN:** 该类实现 `MVImageProcessorV2`，并继承/扩展 `ImageProcessorV2`。 文档字符串指出：Multi-view image processor for Hunyuan3D. 其中较重要的方法包括 `__init__`, `__call__`。

### Lines 1071-1077: Top-level configuration / 顶层配置
```python


# All tool classes available in this module for resolution
TOOL_CLASSES = (
    ImageProcessorV2,
    MVImageProcessorV2,
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1080-1093: Function `resolve_hunyuan3d_tool` / 函数 `resolve_hunyuan3d_tool`
```python
def resolve_hunyuan3d_tool(target: str):
    """Resolve a Hunyuan3D tool class by target string."""
    # First, try to match against _aliases
    for cls in TOOL_CLASSES:
        aliases = getattr(cls, "_aliases", [])
        if target in aliases:
            return cls

    # Then, try to match against class names
    for cls in TOOL_CLASSES:
        if cls.__name__ == target:
            return cls

    return None
```
**EN:** This function drives `resolve_hunyuan3d_tool` with inputs such as `target`. Resolve a Hunyuan3D tool class by target string.
**CN:** 这个函数负责 `resolve_hunyuan3d_tool`，主要处理 `target` 等输入。 文档字符串说明：Resolve a Hunyuan3D tool class by target string.

### Lines 1094-1114: Registration and exports / 注册与导出
```python


__all__ = [
    "transform_pos",
    "get_mv_matrix",
    "get_orthographic_projection_matrix",
    "get_perspective_projection_matrix",
    "export_to_trimesh",
    "mesh_uv_wrap",
    "meshVerticeInpaint",
    "stride_from_shape",
    "scatter_add_nd_with_count",
    "linear_grid_put_2d",
    "MeshRender",
# ...
    "MVImageProcessorV2",
    "TOOL_CLASSES",
    "resolve_hunyuan3d_tool",
]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Command-line interface / 命令行接口
- Image generation flow / 图像生成流程
- 3D/mesh processing / 3D/网格处理
- Symbol `transform_pos` anchors the module API / 符号 `transform_pos` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.csrc.render.mesh_processor`, `sglang.multimodal_gen.csrc.render.hunyuan3d_rasterizer`
- **External / 外部**: `__future__`, `cv2`, `numpy`, `torch`, `torch.nn.functional`, `trimesh`, `einops`, `PIL`
- **Stdlib / 标准库**: `math`, `typing`
