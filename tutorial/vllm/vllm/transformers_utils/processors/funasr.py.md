# funasr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/funasr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for Funasr inputs. / [CN] 为 Funasr 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 5-18: Imports
```python
import numpy as np
import torch
import torch.nn as nn
import torchaudio.compliance.kaldi as kaldi
from torch.nn.utils.rnn import pad_sequence
from transformers import (
    AutoFeatureExtractor,
    BatchFeature,
)
from transformers.feature_extraction_sequence_utils import SequenceFeatureExtractor
from transformers.processing_utils import ProcessorMixin
from transformers.utils import TensorType

from vllm.logger import init_logger
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as none, external APIs such as `numpy`, `torch`, `torch.nn`, `torchaudio.compliance.kaldi`, `torch.nn.utils.rnn`, `transformers`, `transformers.feature_extraction_sequence_utils`, `transformers.processing_utils`, ... (+1 more), and internal vLLM modules such as `vllm.logger`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 无，外部 API 如 `numpy`, `torch`, `torch.nn`, `torchaudio.compliance.kaldi`, `torch.nn.utils.rnn`, `transformers`, `transformers.feature_extraction_sequence_utils`, `transformers.processing_utils`, ... (+1 more)，以及 vLLM 内部模块如 `vllm.logger`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 20-20: Module state and constants
```python
logger = init_logger(__name__)
```
**EN:** This block initializes module-level state such as `logger`. The assigned names act as shared context for the rest of the file.
**CN:** 该代码块初始化了模块级状态，例如 `logger`。这些名称为文件其余部分提供共享上下文。

### Lines 23-37: Function `apply_cmvn`
```python
def apply_cmvn(inputs, cmvn):  # noqa
    """
    Apply CMVN with mvn data
    """

    device = inputs.device
    # dtype = inputs.dtype
    frame, dim = inputs.shape

    means = cmvn[0:1, :dim]
    vars = cmvn[1:2, :dim]
    inputs += means.to(device)
    inputs *= vars.to(device)

    return inputs.type(torch.float32)
```
**EN:** This function implements `apply_cmvn`. The docstring states that Apply CMVN with mvn data Main inputs include `inputs`, `cmvn`.
**CN:** 该函数实现 `apply_cmvn` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `inputs`, `cmvn`。

### Lines 40-60: Function `apply_lfr`
```python
def apply_lfr(inputs, lfr_m, lfr_n):
    # LFR_inputs = []
    T = inputs.shape[0]
    T_lfr = int(np.ceil(T / lfr_n))
    left_padding = inputs[0].repeat((lfr_m - 1) // 2, 1)
    inputs = torch.vstack((left_padding, inputs))
    T = T + (lfr_m - 1) // 2
    feat_dim = inputs.shape[-1]
    strides = (lfr_n * feat_dim, 1)
    sizes = (T_lfr, lfr_m * feat_dim)
    last_idx = (T - lfr_m) // lfr_n + 1
    num_padding = lfr_m - (T - last_idx * lfr_n)
    if num_padding > 0:
        num_padding = (
            (2 * lfr_m - 2 * T + (T_lfr - 1 + last_idx) * lfr_n)
            / 2
            * (T_lfr - last_idx)
        )
        inputs = torch.vstack([inputs] + [inputs[-1:]] * int(num_padding))
    LFR_outputs = inputs.as_strided(sizes, strides)
    return LFR_outputs.clone().type(torch.float32)
```
**EN:** This function implements `apply_lfr`. Main inputs include `inputs`, `lfr_m`, `lfr_n`.
**CN:** 该函数实现 `apply_lfr` 相关逻辑。 主要输入参数包括 `inputs`, `lfr_m`, `lfr_n`。

### Lines 63-86: Function `load_cmvn`
```python
def load_cmvn(cmvn_file):
    with open(cmvn_file, encoding="utf-8") as f:
        lines = f.readlines()
    means_list = []
    vars_list = []
    for i in range(len(lines)):
        line_item = lines[i].split()
        if line_item[0] == "<AddShift>":
            line_item = lines[i + 1].split()
            if line_item[0] == "<LearnRateCoef>":
                add_shift_line = line_item[3 : (len(line_item) - 1)]
                means_list = list(add_shift_line)
                continue
        elif line_item[0] == "<Rescale>":
            line_item = lines[i + 1].split()
            if line_item[0] == "<LearnRateCoef>":
                rescale_line = line_item[3 : (len(line_item) - 1)]
                vars_list = list(rescale_line)
                continue
    means = np.array(means_list).astype(np.float32)
    vars = np.array(vars_list).astype(np.float32)
    cmvn = np.array([means, vars])
    cmvn = torch.as_tensor(cmvn, dtype=torch.float32)
    return cmvn
```
**EN:** This function loads cmvn. Main inputs include `cmvn_file`.
**CN:** 该函数负责完成 `load_cmvn` 对应的核心步骤。 主要输入参数包括 `cmvn_file`。

### Lines 89-218: Class `WavFrontend`
```python
class WavFrontend(nn.Module):
    """Conventional frontend structure for ASR."""

    def __init__(
        self,
        cmvn_file: str = "null",
        fs: int = 16000,
        window: str = "hamming",
        n_mels: int = 80,
        frame_length: int = 25,
        frame_shift: int = 10,
        filter_length_min: int = -1,
        filter_length_max: int = -1,
        lfr_m: int = 1,
        lfr_n: int = 1,
        dither: float = 1.0,
        snip_edges: bool = True,
        upsacle_samples: bool = True,
        **kwargs,
    ):
        super().__init__()
        self.fs = fs
        self.window = window
        self.n_mels = n_mels
        self.frame_length = frame_length
        self.frame_shift = frame_shift
        self.filter_length_min = filter_length_min
        self.filter_length_max = filter_length_max
        self.lfr_m = lfr_m
        self.lfr_n = lfr_n
        self.cmvn_file = cmvn_file
        self.dither = dither
        self.snip_edges = snip_edges
        self.upsacle_samples = upsacle_samples
        self.cmvn = None if self.cmvn_file is None else load_cmvn(self.cmvn_file)

    def output_size(self) -> int:
        return self.n_mels * self.lfr_m

    def forward(
# ... omitted for brevity ...
            mat = input[i, : input_lengths[i], :]
            if self.lfr_m != 1 or self.lfr_n != 1:
                mat = apply_lfr(mat, self.lfr_m, self.lfr_n)
            if self.cmvn is not None:
                mat = apply_cmvn(mat, self.cmvn)
            feat_length = mat.size(0)
            feats.append(mat)
            feats_lens.append(feat_length)

        feats_lens = torch.as_tensor(feats_lens)
        feats_pad = pad_sequence(feats, batch_first=True, padding_value=0.0)
        return feats_pad, feats_lens
```
**EN:** Defines `WavFrontend`, a processor-related class derived from `nn.Module`. The class docstring highlights that Conventional frontend structure for ASR. Key methods include `__init__`, `output_size`, `forward`, `forward_fbank`, `forward_lfr_cmvn`.
**CN:** 定义 `WavFrontend`，这是一个处理器相关类，继承自 `nn.Module`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `output_size`, `forward`, `forward_fbank`, `forward_lfr_cmvn`。

### Lines 221-368: Class `FunASRFeatureExtractor`
```python
class FunASRFeatureExtractor(SequenceFeatureExtractor):
    r"""
    Constructs a FunASR feature extractor.

    This feature extractor inherits from [`~feature_extraction_sequence_
        utils.SequenceFeatureExtractor`] which contains most of the main
        methods. Users should refer to this superclass for more information
        regarding those methods.

    This class extracts mel-filter bank features from raw speech using a custom
    numpy implementation of the `Short Time Fourier Transform` which should
    match pytorch's `torch.stft` equivalent.

    Args:
        feature_size (`int`, *optional*, defaults to 80):
            The feature dimension of the extracted features.
        sampling_rate (`int`, *optional*, defaults to 16000):
            The sampling rate at which the audio files should be digitalized
            expressed in hertz (Hz).
        hop_length (`int`, *optional*, defaults to 160):
            Length of the overlapping windows for the STFT used to obtain the
            Mel Frequency coefficients.
        chunk_length (`int`, *optional*, defaults to 30):
            The maximum number of chunks of `sampling_rate` samples used to
            trim and pad longer or shorter audio sequences.
        n_fft (`int`, *optional*, defaults to 400):
            Size of the Fourier transform.
        padding_value (`float`, *optional*, defaults to 0.0):
            Padding value used to pad the audio. Should correspond to silences.
        dither (`float`, *optional*, defaults to 0.0):
            Adds dithering. In other words, adds a small Gaussian noise to each frame.
            E.g. use 0.0001 to add dithering with a normal distribution centered
            around 0.0 with standard deviation 0.0001 (assuming [-1,+1] range
            of raw_speech). The value 0.0 means no dithering.
            Dithering has similar effect as `spectrogram(mel_floor=...)`. It reduces
            the high log_mel_fbank values for signals with hard-zero sections,
            when VAD cutoff is present in the signal.
    """

    model_input_names = ["input_features"]
# ... omitted for brevity ...
            padding=padding,
            max_length=max_length if max_length else self.max_length,
            truncation=truncation,
            pad_to_multiple_of=pad_to_multiple_of,
            return_attention_mask=return_attention_mask or do_normalize,
        )
        if return_tensors is not None:
            batched_speech = batched_speech.convert_to_tensors(return_tensors)

        batched_speech["speech_lengths"] = torch.tensor(speech_lengths)
        batched_speech["fake_token_lengths"] = torch.concat(fake_token_lengths)
        return batched_speech
```
**EN:** Defines `FunASRFeatureExtractor`, a processor-related class derived from `SequenceFeatureExtractor`. The class docstring highlights that Constructs a FunASR feature extractor. Key methods include `__init__`, `extract_fbank`, `__call__`.
**CN:** 定义 `FunASRFeatureExtractor`，这是一个处理器相关类，继承自 `SequenceFeatureExtractor`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `extract_fbank`, `__call__`。

### Lines 371-480: Class `FunASRProcessor`
```python
class FunASRProcessor(ProcessorMixin):
    r"""
    Constructs a FunASR processor which wraps a FunASR feature extractor and
    a FunASR tokenizer into a single processor.

    [`FunASRProcessor`] offers all the functionalities of
    [`FunASRFeatureExtractor`] and [`Qwen2Tokenizer`]. See the
    [`~FunASRProcessor.__call__`] and [`~FunASRProcessor.decode`] for more
    information.

    Args:
        feature_extractor (`FunASRFeatureExtractor`): An instance of
            [`FunASRFeatureExtractor`].
            The feature extractor is a required input.
        tokenizer (`Qwen2Tokenizer`):
            An instance of [`Qwen2Tokenizer`]. The tokenizer is a required
            input.
    """

    feature_extractor_class = "FunASRFeatureExtractor"
    tokenizer_class = ("Qwen2Tokenizer", "Qwen2TokenizerFast")

    def __init__(
        self,
        feature_extractor,
        tokenizer,
        audio_token="<|AUDIO|>",
    ):
        super().__init__(feature_extractor, tokenizer)
        self.current_processor = self.feature_extractor
        self._in_target_context_manager = False
        self.audio_token = (
            tokenizer.audio_token if hasattr(tokenizer, "audio_token") else audio_token
        )
        self.audio_token_id = tokenizer.convert_tokens_to_ids(self.audio_token)

    def get_decoder_prompt_ids(self, task=None, language=None, no_timestamps=True):
        return self.tokenizer.get_decoder_prompt_ids(
            task=task, language=language, no_timestamps=no_timestamps
        )
# ... omitted for brevity ...
        if text is None:
            return inputs

        elif audio is None:
            return encodings
        else:
            inputs["labels"] = encodings["input_ids"]

            return inputs

    def get_prompt_ids(self, text: str, return_tensors="np"):
        return self.tokenizer.get_prompt_ids(text, return_tensors=return_tensors)
```
**EN:** Defines `FunASRProcessor`, a processor-related class derived from `ProcessorMixin`. The class docstring highlights that Constructs a FunASR processor which wraps a FunASR feature extractor and Key methods include `__init__`, `get_decoder_prompt_ids`, `__call__`, `get_prompt_ids`.
**CN:** 定义 `FunASRProcessor`，这是一个处理器相关类，继承自 `ProcessorMixin`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `get_decoder_prompt_ids`, `__call__`, `get_prompt_ids`。

### Lines 483-483: Import-time call `AutoFeatureExtractor.register`
```python
AutoFeatureExtractor.register("FunASRFeatureExtractor", FunASRFeatureExtractor)
```
**EN:** This import-time call executes `AutoFeatureExtractor.register` immediately so registration, patching, or setup side effects are applied before the module is used.
**CN:** 该导入时调用会立即执行 `AutoFeatureExtractor.register`，确保注册、补丁或初始化副作用在模块被使用前就已经生效。

## Key Concepts / 关键概念
- **EN:** Processor wrappers unify tokenizers and modality-specific preprocessors behind a single serving-oriented interface.
- **CN:** 处理器封装把 tokenizer 与模态专用预处理器统一到面向服务的单一接口之下。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。
- **EN:** Internal vLLM imports connect the file to runtime registries, logging, artifact lookup, and model-execution utilities.
- **CN:** vLLM 内部导入会把该文件连接到运行时注册表、日志、模型文件查找以及模型执行工具。
- **EN:** Torch types and tensor conventions influence preprocessing, dtype selection, or runtime metadata in this module.
- **CN:** Torch 的类型与张量约定会影响本模块中的预处理、数据类型选择或运行时元数据。

## Dependencies / 依赖关系
- **EN:** Standard library modules: none.
- **CN:** 标准库模块：无。
- **EN:** External packages: `numpy`, `torch`, `torch.nn`, `torchaudio.compliance.kaldi`, `torch.nn.utils.rnn`, `transformers`, `transformers.feature_extraction_sequence_utils`, `transformers.processing_utils`, `transformers.utils`.
- **CN:** 外部依赖包：`numpy`, `torch`, `torch.nn`, `torchaudio.compliance.kaldi`, `torch.nn.utils.rnn`, `transformers`, `transformers.feature_extraction_sequence_utils`, `transformers.processing_utils`, `transformers.utils`。
- **EN:** Internal modules: `vllm.logger`.
- **CN:** 内部模块：`vllm.logger`。
