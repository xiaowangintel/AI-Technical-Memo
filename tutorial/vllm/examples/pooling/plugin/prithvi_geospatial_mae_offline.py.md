# prithvi_geospatial_mae_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/plugin/prithvi_geospatial_mae_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates custom pooling plugin integration. / 演示自定义池化插件集成。

## Line-by-Line Analysis / 逐行分析
### Imports
```python
import argparse
import datetime
import os

import albumentations
import numpy as np
import rasterio
import regex as re
import torch
from einops import rearrange
from terratorch.datamodules import Sen1Floods11NonGeoDataModule

from vllm import LLM
```
**EN:** This block loads helper libraries such as argparse, datetime, os, albumentations, and numpy and pulls in vLLM APIs like vllm.
**CN:** 这一部分加载 argparse、datetime、os、albumentations，以及 numpy 等辅助库，并引入 vllm 等 vLLM API。

### Top-level setup
```python
torch.set_default_dtype(torch.float16)

NO_DATA = -9999
NO_DATA_FLOAT = 0.0001
OFFSET = 0
PERCENTILE = 99

datamodule_config = {
    "bands": ["BLUE", "GREEN", "RED", "NIR_NARROW", "SWIR_1", "SWIR_2"],
    "batch_size": 16,
    "constant_scale": 0.0001,
    "data_root": "/dccstor/geofm-finetuning/datasets/sen1floods11",
    "drop_last": True,
    "no_data_replace": 0.0,
    "no_label_replace": -1,
    "num_workers": 8,
    "test_transform": [
        albumentations.Resize(
            always_apply=False, height=448, interpolation=1, p=1, width=448
        ),
        albumentations.pytorch.ToTensorV2(
            transpose_mask=False, always_apply=True, p=1.0
        ),
    ],
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as NO_DATA, NO_DATA_FLOAT, OFFSET, PERCENTILE, and datamodule_config. It also performs early helper calls such as torch.set_default_dtype, albumentations.Resize, and albumentations.pytorch.ToTensorV2.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 NO_DATA、NO_DATA_FLOAT、OFFSET、PERCENTILE，以及 datamodule_config 等变量。它还会提前执行 torch.set_default_dtype、albumentations.Resize，以及 albumentations.pytorch.ToTensorV2 等辅助调用。

### Class: PrithviMAE
```python
class PrithviMAE:
    def __init__(self, model):
        self.model = LLM(
            model=model,
            skip_tokenizer_init=True,
            dtype="float16",
            enforce_eager=True,
            model_impl="terratorch",
            enable_mm_embeds=True,
        )

    def run(self, input_data, location_coords):
        # merge the inputs into one data structure
        if input_data is not None and input_data.dtype == torch.float32:
            input_data = input_data.to(torch.float16)
            input_data = input_data[0]

        mm_data = {
            "image": {
                "pixel_values": input_data,
                "location_coords": location_coords,
            }
        }

        prompt = {"prompt_token_ids": [1], "multi_modal_data": mm_data}
        outputs = self.model.encode(prompt, pooling_task="plugin", use_tqdm=False)

        return outputs[0].outputs.data
```
**EN:** This class packages the PrithviMAE abstraction used by the example. Notable methods include __init__ and run.
**CN:** 该类封装了示例中使用的 PrithviMAE 抽象。较重要的方法包括 __init__ 和 run。

### Function: generate_datamodule
```python
def generate_datamodule():
    datamodule = Sen1Floods11NonGeoDataModule(
        data_root=datamodule_config["data_root"],
        batch_size=datamodule_config["batch_size"],
        num_workers=datamodule_config["num_workers"],
        bands=datamodule_config["bands"],
        drop_last=datamodule_config["drop_last"],
        test_transform=datamodule_config["test_transform"],
    )

    return datamodule
```
**EN:** This function invokes model inference and collects outputs. Key operations include Sen1Floods11NonGeoDataModule. The return value feeds the next stage of the example pipeline.
**CN:** 该函数触发模型推理并收集输出。关键操作包括 Sen1Floods11NonGeoDataModule。其返回值会继续传给示例管线的下一阶段。

### Function: process_channel_group
```python
def process_channel_group(orig_img, channels):
    """
    Args:
        orig_img: torch.Tensor representing original image (reference)
        with shape = (bands, H, W).
        channels: list of indices representing RGB channels.

    Returns:
        torch.Tensor with shape (num_channels, height, width)
        for original image
    """

    orig_img = orig_img[channels, ...]
    valid_mask = torch.ones_like(orig_img, dtype=torch.bool)
    valid_mask[orig_img == NO_DATA_FLOAT] = False

    # Rescale (enhancing contrast)
    max_value = max(3000, np.percentile(orig_img[valid_mask], PERCENTILE))
    min_value = OFFSET

    orig_img = torch.clamp((orig_img - min_value) / (max_value - min_value), 0, 1)

    # No data as zeros
    orig_img[~valid_mask] = 0

    return orig_img
```
**EN:** Args: orig_img: torch.Tensor representing original image (reference) with shape = (bands, H, W).. It works with parameters such as orig_img and channels. Key operations include torch.ones_like, max, torch.clamp, and np.percentile. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 orig_img 和 channels 等参数。关键操作包括 torch.ones_like、max、torch.clamp，以及 np.percentile。其返回值会继续传给示例管线的下一阶段。

### Function: read_geotiff
```python
def read_geotiff(file_path: str):
    """Read all bands from *file_path* and return image + meta info.

    Args:
        file_path: path to image file.

    Returns:
        np.ndarray with shape (bands, height, width)
        meta info dict
    """

    with rasterio.open(file_path) as src:
        img = src.read()
        meta = src.meta
        try:
            coords = src.lnglat()
        except Exception:
            # Cannot read coords
            coords = None

    return img, meta, coords
```
**EN:** Read all bands from *file_path* and return image + meta info.. It works with parameters such as file_path. Key operations include rasterio.open, src.read, and src.lnglat. The return value feeds the next stage of the example pipeline.
**CN:** 该函数加载输入数据并为后续阶段做准备。它会处理 file_path 等参数。关键操作包括 rasterio.open、src.read，以及 src.lnglat。其返回值会继续传给示例管线的下一阶段。

### Function: save_geotiff
```python
def save_geotiff(image, output_path: str, meta: dict):
    """Save multi-band image in Geotiff file.

    Args:
        image: np.ndarray with shape (bands, height, width)
        output_path: path where to save the image
        meta: dict with meta info.
    """

    with rasterio.open(output_path, "w", **meta) as dest:
        for i in range(image.shape[0]):
            dest.write(image[i, :, :], i + 1)

    return
```
**EN:** Save multi-band image in Geotiff file.. It works with parameters such as image, output_path, and meta. Key operations include rasterio.open, range, and dest.write.
**CN:** 该函数封装示例中的可复用步骤。它会处理 image、output_path，以及 meta 等参数。关键操作包括 rasterio.open、range，以及 dest.write。

### Function: _convert_np_uint8
```python
def _convert_np_uint8(float_image: torch.Tensor):
    image = float_image.numpy() * 255.0
    image = image.astype(dtype=np.uint8)

    return image
```
**EN:** This function transforms intermediate values into the required format. It works with parameters such as float_image. Key operations include image.astype and float_image.numpy. The return value feeds the next stage of the example pipeline.
**CN:** 该函数把中间结果转换成所需格式。它会处理 float_image 等参数。关键操作包括 image.astype 和 float_image.numpy。其返回值会继续传给示例管线的下一阶段。

### Function: load_example
```python
def load_example(
    file_paths: list[str],
    mean: list[float] = None,
    std: list[float] = None,
    indices: list[int] | None = None,
):
    """Build an input example by loading images in *file_paths*.

    Args:
        file_paths: list of file paths .
        mean: list containing mean values for each band in the
              images in *file_paths*.
        std: list containing std values for each band in the
             images in *file_paths*.

    Returns:
        np.array containing created example
        list of meta info for each image in *file_paths*
    """

    # ... key logic omitted for brevity ...
                    julian_day = (
                        datetime.datetime.strptime(julian_day, "%m%d")
                        .timetuple()
                        .tm_yday
                    )
                temporal_coords.append([year, julian_day])
        except Exception as e:
            print(f"Could not extract timestamp for {file} ({e})")

    imgs = np.stack(imgs, axis=0)  # num_frames, H, W, C
    imgs = np.moveaxis(imgs, -1, 0).astype("float32")  # C, num_frames, H, W
    imgs = np.expand_dims(imgs, axis=0)  # add batch di

    return imgs, temporal_coords, location_coords, metas
```
**EN:** Build an input example by loading images in *file_paths*.. It works with parameters such as file_paths, mean, std, and indices. Key operations include np.moveaxis, int, match.group, np.stack, and astype. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 file_paths、mean、std，以及 indices 等参数。关键操作包括 np.moveaxis、int、match.group、np.stack，以及 astype。其返回值会继续传给示例管线的下一阶段。

### Function: run_model
```python
def run_model(
    input_data,
    temporal_coords,
    location_coords,
    model,
    datamodule,
    img_size,
    lightning_model=None,
):
    # Reflect pad if not divisible by img_size
    original_h, original_w = input_data.shape[-2:]
    pad_h = (img_size - (original_h % img_size)) % img_size
    pad_w = (img_size - (original_w % img_size)) % img_size
    input_data = np.pad(
        input_data, ((0, 0), (0, 0), (0, 0), (0, pad_h), (0, pad_w)), mode="reflect"
    )

    # Build sliding window

    batch_size = 1
    # ... key logic omitted for brevity ...
        w=img_size,
        b=1,
        c=1,
        h1=h1,
        w1=w1,
    )

    # Cut padded area back to original size
    pred_imgs = pred_imgs[..., :original_h, :original_w]

    # Squeeze (batch size 1)
    pred_imgs = pred_imgs[0]

    return pred_imgs
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as input_data, temporal_coords, location_coords, model, datamodule, and img_size. Key operations include torch.tensor, rearrange, unsqueeze, np.pad, and unfold. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 input_data、temporal_coords、location_coords、model、datamodule，以及 img_size 等参数。关键操作包括 torch.tensor、rearrange、unsqueeze、np.pad，以及 unfold。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(
    data_file: str,
    model: str,
    output_dir: str,
    rgb_outputs: bool,
    input_indices: list[int] = None,
):
    os.makedirs(output_dir, exist_ok=True)

    model_obj = PrithviMAE(model=model)
    datamodule = generate_datamodule()
    img_size = 512  # Size of Sen1Floods11

    input_data, temporal_coords, location_coords, meta_data = load_example(
        file_paths=[data_file],
        indices=input_indices,
    )

    meta_data = meta_data[0]  # only one image

    # ... key logic omitted for brevity ...
    )

    # Save image rgb
    if rgb_outputs:
        name_suffix = os.path.splitext(os.path.basename(data_file))[0]
        rgb_file = os.path.join(
            output_dir,
            f"original_rgb_{name_suffix}.tiff",
        )
        save_geotiff(
            image=_convert_np_uint8(rgb_orig),
            output_path=rgb_file,
            meta=meta_data,
        )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as data_file, model, output_dir, rgb_outputs, and input_indices. Key operations include os.path.join, save_geotiff, _convert_np_uint8, os.path.splitext, and os.path.basename.
**CN:** 该函数编排端到端工作流。它会处理 data_file、model、output_dir、rgb_outputs，以及 input_indices 等参数。关键操作包括 os.path.join、save_geotiff、_convert_np_uint8、os.path.splitext，以及 os.path.basename。

### Entry point
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser("MAE run inference", add_help=False)

    parser.add_argument(
        "--data_file",
        type=str,
        default="./India_900498_S2Hand.tif",
        help="Path to the file.",
    )
    parser.add_argument(
        "--model",
        type=str,
    # ... key logic omitted for brevity ...
        "--rgb_outputs",
        action="store_true",
        help="If present, output files will only contain RGB channels. "
        "Otherwise, all bands will be saved.",
    )
    args = parser.parse_args()

    main(**vars(args))
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parser.add_argument, argparse.ArgumentParser, parser.parse_args, and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parser.add_argument、argparse.ArgumentParser、parser.parse_args，以及 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `datetime`, `os`, `albumentations`, `numpy`, `rasterio`, `regex`, `torch` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `PrithviMAE`, `generate_datamodule`, `process_channel_group`, `read_geotiff`, `save_geotiff`, `_convert_np_uint8`, `load_example`, `run_model` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `torch.set_default_dtype`, `albumentations.Resize`, `albumentations.pytorch.ToTensorV2`, `LLM`, `self.model.encode`, `input_data.to`, `Sen1Floods11NonGeoDataModule`, `torch.ones_like` reveal the main execution path / 这些调用体现了主要执行链路。
