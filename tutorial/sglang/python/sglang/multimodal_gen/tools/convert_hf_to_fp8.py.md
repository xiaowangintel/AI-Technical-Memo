# convert_hf_to_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/tools/convert_hf_to_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This script provides developer tooling for convert hf to fp8, typically for conversion, inspection, or offline preparation. Key symbols include `ceildiv`, `block_fp8`, `channel_fp8`. / 该脚本提供与 convert hf to fp8 相关的开发工具，通常用于转换、检查或离线准备。 关键符号包括 `ceildiv`, `block_fp8`, `channel_fp8`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41: Imports and module setup / 导入与模块初始化
```python
# copied and adapted from Slime
"""
Convert HuggingFace safetensors model to FP8 format for efficient inference.

Example usage:
    # convert FLUX.1-dev transformer to FP8
    python -m sglang.multimodal_gen.tools.convert_hf_to_fp8 \
        --model-dir /path/to/FLUX.1-dev/transformer \
        --save-dir /path/to/FLUX.1-dev/transformer-FP8 \
        --strategy block \
        --block-size 128 128

Options:
    --model-dir MODEL_DIR
# ...
from tqdm import tqdm

FP8_INFO = torch.finfo(torch.float8_e4m3fn)
FP8_MAX, FP8_MIN = FP8_INFO.max, FP8_INFO.min
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 44-45: Function `ceildiv` / 函数 `ceildiv`
```python
def ceildiv(a, b):
    return -(-a // b)
```
**EN:** This function drives `ceildiv` with inputs such as `a`, `b`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `ceildiv`，主要处理 `a`, `b` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 48-79: Function `block_fp8` / 函数 `block_fp8`
```python
def block_fp8(weight, block_size):

    # per block quant
    block_n, block_k = block_size[0], block_size[1]

    shape_0, shape_1 = weight.shape

    n_tiles = ceildiv(shape_0, block_n)
    k_tiles = ceildiv(shape_1, block_k)

    q_weight = F.pad(
        weight,
        (0, k_tiles * block_k - shape_1, 0, n_tiles * block_n - shape_0),
        mode="constant",
# ...
    qweight = qweight[:shape_0, :shape_1].clone().detach()
    scale = scale.squeeze()

    return qweight, scale
```
**EN:** This function drives `block_fp8` with inputs such as `weight`, `block_size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `block_fp8`，主要处理 `weight`, `block_size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 82-87: Function `channel_fp8` / 函数 `channel_fp8`
```python
def channel_fp8(weight):
    channel_max = torch.max(weight.abs(), dim=-1, keepdim=True)[0]
    scale = channel_max.clamp(min=1e-12).to(torch.float32) / FP8_MAX
    qweight = (weight / scale).clamp(min=FP8_MIN, max=FP8_MAX)
    qweight = qweight.to(torch.float8_e4m3fn)
    return qweight, scale
```
**EN:** This function drives `channel_fp8` with inputs such as `weight`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `channel_fp8`，主要处理 `weight` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 90-95: Function `tensor_fp8` / 函数 `tensor_fp8`
```python
def tensor_fp8(weight):
    scale = weight.abs().max().clamp(min=1e-12).to(torch.float32) / FP8_MAX
    qweight = (weight / scale).clamp(min=FP8_MIN, max=FP8_MAX)
    qweight = qweight.to(torch.float8_e4m3fn)
    scale = scale.view(1)
    return qweight, scale
```
**EN:** This function drives `tensor_fp8` with inputs such as `weight`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `tensor_fp8`，主要处理 `weight` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 98-104: Function `quant_fp8` / 函数 `quant_fp8`
```python
def quant_fp8(weight, strategy, block_size=None):
    if strategy == "tensor":
        return tensor_fp8(weight)
    elif strategy == "channel":
        return channel_fp8(weight)
    else:
        return block_fp8(weight, block_size)
```
**EN:** This function drives `quant_fp8` with inputs such as `weight`, `strategy`, `block_size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `quant_fp8`，主要处理 `weight`, `strategy`, `block_size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 107-119: Class `ConversionResult` / 类 `ConversionResult`
```python
class ConversionResult:
    def __init__(self):
        self.lock = threading.Lock()
        self.weight_map = {}
        self.param_count = 0
        self.modules_to_not_convert = []

    def add_result(self, filename, q_weights, module_names):
        with self.lock:
            for k, v in q_weights.items():
                self.weight_map[k] = filename
                self.param_count += v.numel()
            self.modules_to_not_convert.extend(module_names)
```
**EN:** This class models `ConversionResult`. Important methods include `__init__`, `add_result`.
**CN:** 该类实现 `ConversionResult`。 其中较重要的方法包括 `__init__`, `add_result`。

### Lines 122-177: Function `process_file` / 函数 `process_file`
```python
def process_file(
    input_path, output_path, filename, strategy, block_size, result_collector
):
    if not filename.endswith(".safetensors"):
        return

    print(f"Processing {filename}, memory usage: {torch.cuda.memory_allocated()}")
    weights = {}
    q_weights = {}

    with safetensors.safe_open(
        os.path.join(input_path, filename), framework="pt", device="cuda"
    ) as f:
        for k in f.keys():
# ...
        q_weights, os.path.join(output_path, filename), metadata={"format": "pt"}
    )

    result_collector.add_result(filename, q_weights, modules_to_not_convert)
```
**EN:** This function drives `process_file` with inputs such as `input_path`, `output_path`, `filename`, `strategy`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `process_file`，主要处理 `input_path`, `output_path`, `filename`, `strategy` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 180-283: Function `convert_fp8` / 函数 `convert_fp8`
```python
def convert_fp8(input_path, output_path, strategy, block_size=None, max_workers=4):
    input_path = os.path.abspath(input_path)
    os.makedirs(output_path, exist_ok=True)

    for filename in os.listdir(input_path):
        if not filename.endswith(".safetensors") and not os.path.isdir(
            os.path.join(input_path, filename)
        ):
            shutil.copyfile(
                os.path.join(input_path, filename), os.path.join(output_path, filename)
            )

    safetensors_files = [
        f for f in os.listdir(input_path) if f.endswith(".safetensors")
# ...
    )

    gc.collect()
    torch.cuda.empty_cache()
```
**EN:** This function drives `convert_fp8` with inputs such as `input_path`, `output_path`, `strategy`, `block_size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `convert_fp8`，主要处理 `input_path`, `output_path`, `strategy`, `block_size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 284-320: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--model-dir",
        type=str,
        help="Path to the directory of the HF safetensors model.",
    )
    parser.add_argument(
        "--save-dir",
        type=str,
        help="Path to the directory to save the converted model.",
    )
# ...

    convert_fp8(
        args.model_dir, args.save_dir, args.strategy, args.block_size, args.max_workers
    )
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- HTTP route definition / HTTP 路由定义
- PyTorch tensor computation / PyTorch 张量计算
- Quantization workflow / 量化工作流
- Caching strategy / 缓存策略
- Symbol `ceildiv` anchors the module API / 符号 `ceildiv` 构成该模块的核心 API

## Dependencies / 依赖关系
- **External / 外部**: `safetensors`, `safetensors.torch`, `torch`, `torch.nn.functional`, `tqdm`
- **Stdlib / 标准库**: `argparse`, `gc`, `json`, `os`, `shutil`, `threading`, `concurrent.futures`
