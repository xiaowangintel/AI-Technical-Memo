# phi4mm_audio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/phi4mm_audio.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the phi4mm audio multimodal runtime stack in SGLang, including model blocks, adapters, and inference helpers. / 该模块实现 SGLang 中 phi4mm audio 的多模态运行时堆栈，包括模型模块、适配器与推理辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 15-41: Module imports
```python
import abc
import math
from typing import Literal, Optional

import numpy as np
import torch
import torch.nn.functional as F
from torch import Tensor, nn
from torch.distributed.algorithms._checkpoint.checkpoint_wrapper import (
    CheckpointWrapper,
)
from torch.distributed.fsdp.fully_sharded_data_parallel import FullyShardedDataParallel
from transformers import PretrainedConfig

from sglang.srt.models.phi4mm_utils import (
    AbsolutePositionalEncoding,
    ConvModule,
    FeedForward,
    MeanVarianceNormLayer,
    MultiHeadedAttention,
    MultiSequential,
    NemoConvSubsampling,
    T5RelativeAttentionLogitBias,
    adaptive_enc_mask,
    get_offset,
    unfold_tensor,
)
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 43-43: Top-level assign
```python
_AUDIO_PLACEHOLDER_TOKEN_ID = 200011  # <|endoftext11|>
```
**EN:** Defines or updates _AUDIO_PLACEHOLDER_TOKEN_ID, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _AUDIO_PLACEHOLDER_TOKEN_ID，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 46-147: Class `ConformerEncoderLayer` overview
```python
class ConformerEncoderLayer(nn.Module):
    """ConformerEncoder Layer module.
    for more details see conformer paper:
        https://arxiv.org/abs/2005.08100
    This module implement the Conformer block layer.

    Args:
        d_model: int
            attention dim.
        ext_pw_out_channel: int
            if > 0, ext_pw_out_channel is a dim channel size
             for the last pointwise conv after swish activation.
        depthwise_seperable_out_channel: int
            if set different to 0, the number of
             depthwise_seperable_out_channel will be used as a
             channel_out of the second conv1d layer.
             otherwise, it equal to 0, the second conv1d layer is skipped.
        depthwise_multiplier: int
            number of input_dim channels duplication. this value
             will be used to compute the hidden channels of the Conv1D.
        n_head: int
            the number of heads for multihead attention module.
        d_ffn: int
            output size of the feed_forward blocks.
        ext_pw_kernel_size: int
            kernel size of the conv pointwise of the conformer.
        kernel_size: int
            kernel size.
        dropout_rate: float
            dropout rate.
        causal: bool, optional
            if set to True, convolution have no access
             to future frames. default False.
        batch_norm: bool, optional
            if set to True, apply batchnorm before activation
            in ConvModule layer of the conformer.
            default False
        activation: str, optional
            activation function name,
            one of ["relu", "swish", "sigmoid"],
# ... truncated for brevity ...
```
**EN:** Defines `ConformerEncoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ConformerEncoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 148-223: Method `ConformerEncoderLayer.__init__`
```python
    def __init__(
        self,
        d_model=512,
        ext_pw_out_channel=0,
        depthwise_seperable_out_channel=256,
        depthwise_multiplier=1,
        n_head=4,
        d_ffn=2048,
        ext_pw_kernel_size=1,
        kernel_size=3,
        dropout_rate=0.1,
        causal=False,
        batch_norm=False,
        activation="relu",
        chunk_se=0,
        chunk_size=18,
        conv_activation="relu",
        conv_glu_type="sigmoid",
        bias_in_glu=True,
        linear_glu_in_convm=False,
        attention_inner_dim=-1,
        attention_glu_type="swish",
        activation_checkpointing="",
        export=False,
        use_pt_scaled_dot_product_attention=False,
        attn_group_sizes: int = 1,
    ):
        super().__init__()

        self.feed_forward_in = FeedForward(
            d_model=d_model,
            d_inner=d_ffn,
            dropout_rate=dropout_rate,
            activation=activation,
            bias_in_glu=bias_in_glu,
        )

        self.self_attn = MultiHeadedAttention(
            n_head,
            d_model,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(d_model=..., ext_pw_out_channel=..., depthwise_seperable_out_channel=..., depthwise_multiplier=..., n_head=..., d_ffn=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(d_model=..., ext_pw_out_channel=..., depthwise_seperable_out_channel=..., depthwise_multiplier=..., n_head=..., d_ffn=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 225-263: Method `ConformerEncoderLayer.forward`
```python
    def forward(
        self,
        x,
        pos_k,
        pos_v,
        mask,
        relative_attention_bias: Optional[Tensor] = None,
    ):
        """ConformerEncoder forward.

        Args:
            x: torch.Tensor
                input feature of shape (batch, max_time_in, size)
            pos_k: torch.Tensor
                positional key embedding.
            mask: torch.Tensor
                mask for x (batch, max_time_in)
            relative_attention_bias: Optional[torch.Tensor]
                bias added to attention logits w.r.t. relative positions
                (1, n_head, time1, time2)
        """
        x = x + 0.5 * self.feed_forward_in(x)
        norm_x = self.layer_norm_att(x)

        x = x + self.self_attn(
            norm_x,
            norm_x,
            norm_x,
            pos_k,
            pos_v,
            mask,
            relative_attention_bias=relative_attention_bias,
        )
        x = x + self.conv(x)
        x = x + 0.5 * self.feed_forward_out(x)

        out = self.layer_norm(x)

        return out, pos_k, pos_v, mask
```
**EN:** This method implements `forward(x, pos_k, pos_v, mask, relative_attention_bias: ...=...)` and ConformerEncoder forward.
**CN:** 这个方法实现了 `forward(x, pos_k, pos_v, mask, relative_attention_bias: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 266-338: Class `TransformerEncoderBase` overview
```python
class TransformerEncoderBase(abc.ABC, nn.Module):
    """The Base class for Transformer based encoders

    Please set causal = True in streaming model
    Args:
        input_size: int
            input feature dimension.
        chunk_size: int, list(int)
            Number of frames for each chunk
            This variable can take 2 forms:
            int:  Used for inference, or single chunk size training
            list(int) : Used only for variable chunk size training
            Some examples for the 2 cases:
            chunk_size = 12
            chunk_size = [6, 8, 12, 24]
        left_chunk: int, list(int)
            Number of chunks used for masking in streaming mode.
            This variable can take 2 forms:
            int:  Used for inference, or single chunk size training
            list(int) : Used only for variable chunk size training. When
            chunk_size is a list, left_chunk must be a list with same length.
            Some examples for the 2 cases:
            left_chunk = 6
            left_chunk = [12, 9, 6, 3]
        attention_dim: int, optional
            attention dimension. default 256.
        attention_heads: int, optional
            the number of heads. default 4
        input_layer: str, optional
            input layer type before Conformer,
            one of ["linear", "conv2d", "custom", "vgg2l", "embed"],
            default "conv2d"
        cnn_out: int, optional
            the number of CNN channels before Conformer.
            default -1.
        cnn_layer_norm: bool, optional
            layer norm between Conformer and the first CNN.
            default False.
        time_reduction: int, optional
            time reduction factor
# ... truncated for brevity ...
```
**EN:** Defines `TransformerEncoderBase` as a reusable runtime type derived from abc.ABC, nn.Module. The class groups 10 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformerEncoderBase`，其继承关系为 abc.ABC, nn.Module。这个类组织了 10 个方法，用于实现模型相关行为。

### Lines 339-421: Method `TransformerEncoderBase.__init__`
```python
    def __init__(
        self,
        input_size,
        chunk_size,
        left_chunk,
        attention_dim=256,
        attention_heads=4,
        input_layer="nemo_conv",
        cnn_out=-1,
        cnn_layer_norm=False,
        time_reduction=4,
        dropout_rate=0.0,
        padding_idx=-1,
        relative_attention_bias_args=None,
        positional_dropout_rate=0.0,
        nemo_conv_settings=None,
        conv2d_extra_padding: Literal["feat", "feat_time", "none", True] = "none",
        attention_group_size=1,
        encoder_embedding_config=None,
    ):
        super().__init__()
        self.input_size = input_size
        self.input_layer = input_layer
        self.chunk_size = chunk_size
        self.left_chunk = left_chunk
        self.attention_dim = attention_dim
        self.num_heads = attention_heads
        self.attention_group_size = attention_group_size
        self.time_reduction = time_reduction
        self.nemo_conv_settings = nemo_conv_settings
        self.encoder_embedding_config = encoder_embedding_config

        if self.input_layer == "nemo_conv":
            default_nemo_conv_settings = {
                "subsampling": "dw_striding",
                "subsampling_factor": self.time_reduction,
                "feat_in": input_size,
                "feat_out": attention_dim,
                "conv_channels": 256,
                "subsampling_conv_chunking_factor": 1,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(input_size, chunk_size, left_chunk, attention_dim=..., attention_heads=..., input_layer=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(input_size, chunk_size, left_chunk, attention_dim=..., attention_heads=..., input_layer=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 423-455: Method `TransformerEncoderBase.compute_lens_change`
```python
    def compute_lens_change(self, feature_lens):
        """feature_lens: int
        return updated feature lens.

        This used to return a different lambda function for each case that
        computed the right thing.  That does not work within Torchscript.
        If you really need this to be faster, create nn.Module()-s for all
        the cases and return one of them.  Torchscript does support that.
        """
        if self.input_layer == "nemo_conv":
            # Handle the special causal case
            subsampling_causal_cond = self.nemo_conv_settings.get(
                "subsampling", "dw_striding"
            ) in [
                "dw_striding",
                "striding",
                "striding_conv1d",
            ]
            is_causal = self.nemo_conv_settings.get("is_causal", False)
            if is_causal and subsampling_causal_cond:
                lens_change = (
                    torch.ceil(feature_lens / self.time_reduction).long()
                    if isinstance(feature_lens, Tensor)
                    else math.ceil(feature_lens / self.time_reduction)
                )
                feature_lens_remainder = feature_lens % self.time_reduction
                if isinstance(feature_lens, Tensor):
                    lens_change[feature_lens_remainder != 1] += 1
                elif feature_lens_remainder != 1:
                    lens_change += 1
                return lens_change
            ceil_func = math.ceil if isinstance(feature_lens, int) else torch.ceil
            return ceil_func(feature_lens / self.time_reduction)
```
**EN:** This method implements `compute_lens_change(feature_lens)` and feature_lens: int.
**CN:** 这个方法实现了 `compute_lens_change(feature_lens)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 457-459: Method `TransformerEncoderBase.forward`
```python
    @abc.abstractmethod
    def forward(self):
        """Abstract forward method implementation."""
```
**EN:** This method implements `forward()` and Abstract forward method implementation. Decorators: abc.abstractmethod.
**CN:** 这个方法实现了 `forward()`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：abc.abstractmethod。

### Lines 461-488: Method `TransformerEncoderBase._chunk_size_selection`
```python
    def _chunk_size_selection(self, chunk_size=None, left_chunk=None):
        """If chunk size is a list, we will randomly select a chunk size."""

        if chunk_size is None:
            chunk_size = self.chunk_size
        if left_chunk is None:
            left_chunk = self.left_chunk
        if isinstance(chunk_size, list):
            # Variable chunk size during training
            chunk_size_index = int(
                torch.randint(low=0, high=len(chunk_size), size=(1,))
            )
            chunk_size_train_eff = chunk_size[chunk_size_index]
            if not isinstance(left_chunk, list):
                raise ValueError(
                    "Since chunk_size is a list, left_chunk must be a list"
                )
            if len(left_chunk) != len(chunk_size):
                raise ValueError(
                    "The length of left_chunk must be the same as length of "
                    "chunk_size."
                )
            left_chunk_train_eff = left_chunk[chunk_size_index]
        else:
            chunk_size_train_eff = chunk_size
            left_chunk_train_eff = left_chunk

        return chunk_size_train_eff, left_chunk_train_eff
```
**EN:** This method implements `_chunk_size_selection(chunk_size=..., left_chunk=...)` and If chunk size is a list, we will randomly select a chunk size.
**CN:** 这个方法实现了 `_chunk_size_selection(chunk_size=..., left_chunk=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 490-499: Method `TransformerEncoderBase._get_embed_class`
```python
    def _get_embed_class(self, embed):
        # pylint: disable=protected-access
        is_embed_using_act_chkpt = isinstance(embed, CheckpointWrapper)
        is_embed_fsdp_wrapped = isinstance(embed, FullyShardedDataParallel)
        embed_class = embed
        if is_embed_using_act_chkpt:
            embed_class = embed._checkpoint_wrapped_module
        if is_embed_fsdp_wrapped:
            embed_class = embed.module
        return embed_class
```
**EN:** This method implements `_get_embed_class(embed)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `_get_embed_class(embed)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 501-505: Method `TransformerEncoderBase._forward_embeddings_core`
```python
    def _forward_embeddings_core(self, input_tensor, masks):
        embed_class = self._get_embed_class(self.embed)
        assert isinstance(embed_class, NemoConvSubsampling)
        input_tensor, masks = self.embed(input_tensor, masks)
        return input_tensor, masks
```
**EN:** This method implements `_forward_embeddings_core(input_tensor, masks)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_embeddings_core(input_tensor, masks)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 507-514: Method `TransformerEncoderBase._position_embedding`
```python
    def _position_embedding(self, input_tensor):
        pos_k = None
        pos_v = None
        if self.relative_attention_bias_layer is None:
            input_tensor = self.pos_emb(
                input_tensor
            )  # default to add abs sinusoid embedding
        return pos_k, pos_v
```
**EN:** This method implements `_position_embedding(input_tensor)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `_position_embedding(input_tensor)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 516-532: Method `TransformerEncoderBase._streaming_mask`
```python
    def _streaming_mask(self, seq_len, batch_size, chunk_size, left_chunk):
        chunk_size_train_eff, left_chunk_train_eff = self._chunk_size_selection(
            chunk_size, left_chunk
        )

        # Create mask matrix for streaming
        # S stores start index. if chunksize is 18, s is [0,18,36,....]
        chunk_start_idx = np.arange(0, seq_len, chunk_size_train_eff)

        enc_streaming_mask = (
            adaptive_enc_mask(
                seq_len, chunk_start_idx, left_window=left_chunk_train_eff
            )
            .unsqueeze(0)
            .expand([batch_size, -1, -1])
        )
        return enc_streaming_mask
```
**EN:** This method implements `_streaming_mask(seq_len, batch_size, chunk_size, left_chunk)` and implements a focused helper that supports the surrounding runtime flow inside `TransformerEncoderBase`.
**CN:** 这个方法实现了 `_streaming_mask(seq_len, batch_size, chunk_size, left_chunk)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformerEncoderBase` 内部调用。

### Lines 534-596: Method `TransformerEncoderBase.forward_embeddings`
```python
    def forward_embeddings(self, xs_pad, masks, chunk_size_nc=None, left_chunk_nc=None):
        """Forwarding the inputs through the top embedding layers

        Args:
            xs_pad: torch.Tensor
                input tensor
            masks: torch.Tensor
                input mask
            chunk_size_nc: (optional, default is None) chunk size for
                            non-causal layers
            left_chunk_nc: (optional, default is None) # of left chunks for
                            non-causal layers
        """
        # pylint: disable=R0915
        # get new lens.
        seq_len = int(self.compute_lens_change(xs_pad.shape[1]))
        if seq_len <= 0:
            raise ValueError(
                f"""The sequence length after time reduction is invalid:
                {seq_len}. Your input feature is too short. Consider
                filtering out the very short sentence from data
                loader""",
            )

        batch_size = xs_pad.shape[0]

        enc_streaming_mask = self._streaming_mask(
            seq_len, batch_size, self.chunk_size, self.left_chunk
        )

        if xs_pad.is_cuda:
            enc_streaming_mask = enc_streaming_mask.cuda()
            xs_pad = xs_pad.cuda()

        input_tensor = xs_pad
        input_tensor, masks = self._forward_embeddings_core(input_tensor, masks)

        streaming_mask = enc_streaming_mask
        if streaming_mask is not None and masks is not None:
            hs_mask = masks & streaming_mask
# ... truncated for brevity ...
```
**EN:** This method implements `forward_embeddings(xs_pad, masks, chunk_size_nc=..., left_chunk_nc=...)` and Forwarding the inputs through the top embedding layers.
**CN:** 这个方法实现了 `forward_embeddings(xs_pad, masks, chunk_size_nc=..., left_chunk_nc=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 598-605: Method `TransformerEncoderBase.get_offset`
```python
    def get_offset(self):
        """Returns offset used when retaining inputs for decoding.

        This is essentially, how many additional frames have to be added to
        the front-end CNN input to ensure it can produce a single output.
        So if the "padding" parameter is 0, typically offset will be > 0.
        """
        return get_offset(self.input_layer, self.time_reduction)
```
**EN:** This method implements `get_offset()` and Returns offset used when retaining inputs for decoding.
**CN:** 这个方法实现了 `get_offset()`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 608-775: Class `ConformerEncoder` overview
```python
class ConformerEncoder(TransformerEncoderBase):
    """ConformerEncoder module.
    see original paper for more details:
        https://arxiv.org/abs/2005.08100

    Please set causal = True in streaming model
    Args:
        input_size: int
            input feature dimension.
        chunk_size: int, list(int)
            Number of frames for each chunk
            This variable can take 2 forms:
            int:  Used for inference, or single chunk size training
            list(int) : Used only for variable chunk size training
            Some examples for the 2 cases:
            chunk_size = 12
            chunk_size = [6, 8, 12, 24]
        left_chunk: int, list(int)
            Number of chunks used for masking in streaming mode.
            This variable can take 2 forms:
            int:  Used for inference, or single chunk size training
            list(int) : Used only for variable chunk size training. When
            chunk_size is a list, left_chunk must be a list with same length.
            Some examples for the 2 cases:
            left_chunk = 6
            left_chunk = [12, 9, 6, 3]
        left_chunk: int
            number of chunks used for masking in streaming mode.
        num_lang: int
            This parameter is used to store the number of languages in the
            lang_dict, only used for multiseed/multilingual models.
            default None.
        attention_dim: int, optional
            attention dimension. default 256.
        attention_heads: int, optional
            the number of heads. default 4
        linear_units:
            the number of units of position-wise feed forward.
            default 2048
        num_block:
# ... truncated for brevity ...
```
**EN:** Defines `ConformerEncoder` as a reusable runtime type derived from TransformerEncoderBase. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ConformerEncoder`，其继承关系为 TransformerEncoderBase。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 776-776: Class `ConformerEncoder` attributes
```python
    extra_multi_layer_output_idxs: list[int]
```
**EN:** Defines class-level attributes and metadata that shape how `ConformerEncoder` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `ConformerEncoder` 在运行时的行为。

### Lines 778-882: Method `ConformerEncoder.__init__`
```python
    def __init__(  # pylint: disable-all
        self,
        input_size,
        chunk_size,
        left_chunk,
        num_lang=None,
        attention_dim=256,
        attention_heads=4,
        linear_units=2048,
        num_blocks=6,
        dropout_rate=0.1,
        input_layer="nemo_conv",
        causal=True,
        batch_norm=False,
        cnn_out=-1,
        cnn_layer_norm=False,
        ext_pw_out_channel=0,
        ext_pw_kernel_size=1,
        depthwise_seperable_out_channel=256,
        depthwise_multiplier=1,
        chunk_se=0,
        kernel_size=3,
        activation="relu",
        conv_activation="relu",
        conv_glu_type="sigmoid",
        bias_in_glu=True,
        linear_glu_in_convm=False,
        attention_glu_type="swish",
        export=False,
        extra_layer_output_idx=-1,
        extra_multi_layer_output_idxs=[],  # noqa
        activation_checkpointing="",
        relative_attention_bias_args=None,
        time_reduction=4,
        use_pt_scaled_dot_product_attention=False,
        nemo_conv_settings=None,
        conv2d_extra_padding: Literal["feat", "feat_time", "none", True] = "none",
        replication_pad_for_subsample_embedding=False,
        attention_group_size=1,
        encoder_embedding_config=None,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(input_size, chunk_size, left_chunk, num_lang=..., attention_dim=..., attention_heads=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(input_size, chunk_size, left_chunk, num_lang=..., attention_dim=..., attention_heads=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 884-886: Method `ConformerEncoder.init_relative_attention_bias`
```python
    def init_relative_attention_bias(self, input_tensor):
        if self.relative_attention_bias_layer:
            return self.relative_attention_bias_layer(input_tensor)
```
**EN:** This method implements `init_relative_attention_bias(input_tensor)` and implements attention-related tensor preparation or execution.
**CN:** 这个方法实现了 `init_relative_attention_bias(input_tensor)`，其作用是实现与注意力相关的张量准备或执行逻辑。

### Lines 888-905: Method `ConformerEncoder.calculate_hs_mask`
```python
    def calculate_hs_mask(self, xs_pad, device, mask):
        max_audio_length = xs_pad.shape[1]
        batch_size = xs_pad.shape[0]
        enc_streaming_mask = self._streaming_mask(
            max_audio_length, batch_size, self.chunk_size, self.left_chunk
        )
        enc_streaming_mask = enc_streaming_mask.to(device)
        if mask is None:
            return enc_streaming_mask

        feature_lens = mask.sum(1)
        padding_length = feature_lens
        pad_mask = torch.arange(0, max_audio_length, device=device).expand(
            padding_length.size(0), -1
        ) < padding_length.unsqueeze(1)
        pad_mask = pad_mask.unsqueeze(1)
        pad_mask = pad_mask & enc_streaming_mask
        return pad_mask
```
**EN:** This method implements `calculate_hs_mask(xs_pad, device, mask)` and implements a focused helper that supports the surrounding runtime flow inside `ConformerEncoder`.
**CN:** 这个方法实现了 `calculate_hs_mask(xs_pad, device, mask)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ConformerEncoder` 内部调用。

### Lines 907-995: Method `ConformerEncoder.forward`
```python
    @torch.jit.ignore
    def forward(self, xs_pad, masks):
        """Conformer Forward function

        Args:
            xs_pad: torch.Tensor
                input tensor
            masks: torch.Tensor
                post-embedding input lengths
        """
        xs_pad = self.encoder_embedding(xs_pad)
        input_tensor, pos_k, pos_v, hs_mask, masks = self.forward_embeddings(
            xs_pad, masks
        )

        unfolded = False
        ori_bz, seq_len, D = input_tensor.shape
        max_seq_len = 500  # maximum position for absolute positional encoding
        if seq_len > max_seq_len:
            # audio sequence is longer than max_seq_len, unfold it into chunks
            # of max_seq_len
            unfolded = True
            # the unfold op will drop residual frames, pad it to the multiple
            # of max_seq_len
            if seq_len % max_seq_len > 0:
                chunk_pad_size = max_seq_len - (seq_len % max_seq_len)
            else:
                chunk_pad_size = 0
            if chunk_pad_size > 0:
                input_tensor_pad = F.pad(
                    input_tensor, (0, 0, 0, chunk_pad_size), "constant", 0
                )
                input_tensor = input_tensor_pad.to(input_tensor.device)
            input_tensor = unfold_tensor(input_tensor, max_seq_len)
            if masks is not None:
                # revise hs_mask here because the previous calculated hs_mask
                # did not consider extra pad
                subsampled_pad_mask = masks.squeeze(
                    1
                )  # [bz, subsampled_unmask_seq_len]
# ... truncated for brevity ...
```
**EN:** This method implements `forward(xs_pad, masks)` and Conformer Forward function. Decorators: torch.jit.ignore.
**CN:** 这个方法实现了 `forward(xs_pad, masks)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：torch.jit.ignore。

### Lines 998-1000: Class `WindowQformer` overview
```python
class WindowQformer(nn.Module):
    """Window-level Qformer"""
```
**EN:** Defines `WindowQformer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `WindowQformer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 1001-1033: Method `WindowQformer.__init__`
```python
    def __init__(
        self,
        window_size: int = 8,
        num_queries: int = 1,
        num_blocks: int = 2,
        attention_dim: int = 512,
        attention_heads: int = 8,
        linear_units: int = 2048,
        dropout_rate: float = 0.0,
        normalize_before: bool = True,
    ):
        super().__init__()

        self.decoders = nn.ModuleList(
            [
                nn.TransformerDecoderLayer(
                    d_model=attention_dim,
                    nhead=attention_heads,
                    dim_feedforward=linear_units,
                    dropout=dropout_rate,
                    activation="relu",
                    batch_first=True,
                    norm_first=normalize_before,  # TODO need to verify
                )
                for _ in range(num_blocks)
            ]
        )

        self.queries = nn.Parameter(torch.zeros(1, num_queries, attention_dim))
        self.after_norm = (
            nn.LayerNorm(attention_dim, eps=1e-12) if normalize_before else None
        )
        self.window_size = window_size
```
**EN:** This method implements `__init__(window_size: ...=..., num_queries: ...=..., num_blocks: ...=..., attention_dim: ...=..., attention_heads: ...=..., linear_units: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(window_size: ...=..., num_queries: ...=..., num_blocks: ...=..., attention_dim: ...=..., attention_heads: ...=..., linear_units: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1035-1072: Method `WindowQformer.forward`
```python
    def forward(self, audio_embed, mask, embed_len=None):
        """forward decoder"""
        # audio_embed: N x T x D => N x D x T

        audio_embed = audio_embed.transpose(1, 2)
        # audio_embed: N x D x 1 x T => N x DK x T'
        padding = audio_embed.shape[-1] % self.window_size
        if padding > 0:
            audio_embed = F.pad(
                audio_embed, (0, self.window_size - padding), "constant", 0
            )

        embed_chunk = F.unfold(
            audio_embed[..., None, :],
            kernel_size=(1, self.window_size),
            stride=(1, self.window_size),
        )
        bsz, _, slen = embed_chunk.shape
        # N x D x K x T'
        embed_chunk = embed_chunk.view(bsz, -1, self.window_size, slen)
        # N x T' x K x D
        embed_chunk = embed_chunk.transpose(1, 3).contiguous()
        # NT' x K x D
        embed_chunk = embed_chunk.view(bsz * slen, self.window_size, -1)
        # NT' x 1 x D
        q = self.queries.expand(bsz * slen, -1, -1)
        for layer in self.decoders:
            q = layer(tgt=q, memory=embed_chunk, tgt_mask=None, memory_mask=mask)

        if self.after_norm is not None:
            q = self.after_norm(q)

        if embed_len is not None:
            embed_len = embed_len // self.window_size
        # N x T' x D
        out = q.view(bsz, slen, -1)

        return out, embed_len
```
**EN:** This method implements `forward(audio_embed, mask, embed_len=...)` and forward decoder.
**CN:** 这个方法实现了 `forward(audio_embed, mask, embed_len=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1075-1077: Class `AudioEmbedding` overview
```python
class AudioEmbedding(nn.Module):
    """Image embedding."""
```
**EN:** Defines `AudioEmbedding` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AudioEmbedding`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 1078-1180: Method `AudioEmbedding.__init__`
```python
    def __init__(self, config: PretrainedConfig, **kwargs) -> None:
        super().__init__()
        self.config = config
        # n_embed or hidden_size for text LM
        hidden_size = config.n_embd if hasattr(config, "n_embd") else config.hidden_size

        # self.wte = nn.Embedding(config.vocab_size, hidden_size)

        audio_dim_out = (
            None  # Set this variable according to the actual audio processor
        )
        self.layer_idx = -2

        if (
            isinstance(config.audio_processor, dict)
            and config.audio_processor.get("name", None) == "cascades"
        ):
            encoder_config = config.audio_processor.get("config", None)
            assert encoder_config is not None
            self.encoder = ConformerEncoder(**encoder_config)

            audio_dim_out = encoder_config["attention_dim"]
            n_mels = encoder_config["input_size"]
        else:
            raise NotImplementedError("")

        assert audio_dim_out is not None, "Remember to set values for audio_dim_out"
        self.audio_dim_out = audio_dim_out
        self.audio_dim_in = n_mels

        self.freeze_audio_processor = kwargs.get("freeze_audio_processor", False)

        self.downsample_rate = kwargs.get("downsample_rate", 1)

        if kwargs.get("use_qformer", False):
            qformer_config = kwargs.get("qformer_config", {})
            qformer_config["attention_dim"] = audio_dim_out
            self.qformer = WindowQformer(**qformer_config)
        else:
            self.qformer = None
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1182-1183: Method `AudioEmbedding.set_audio_embeds`
```python
    def set_audio_embeds(self, input_embeds: torch.FloatTensor) -> None:
        self.input_embeds = input_embeds
```
**EN:** This method implements `set_audio_embeds(input_embeds: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_audio_embeds(input_embeds: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1185-1186: Method `AudioEmbedding.set_audio_embed_sizes`
```python
    def set_audio_embed_sizes(self, audio_embed_sizes: torch.LongTensor) -> None:
        self.audio_embed_sizes = audio_embed_sizes
```
**EN:** This method implements `set_audio_embed_sizes(audio_embed_sizes: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_audio_embed_sizes(audio_embed_sizes: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1188-1240: Method `AudioEmbedding.get_audio_features`
```python
    def get_audio_features(
        self,
        input_embeds: torch.FloatTensor,
        audio_attention_mask: torch.Tensor = None,
        audio_projection_mode: str = "speech",
    ) -> torch.FloatTensor:
        """
        arguments:
            input_embeds: audio features (B, T, D)  B: num audios in a sequence
        """
        if self.freeze_audio_processor:
            with torch.no_grad():
                audio_features, masks = self.encoder(input_embeds, audio_attention_mask)
        else:
            audio_features, masks = self.encoder(input_embeds, audio_attention_mask)

        if self.qformer is not None:
            audio_features, _ = self.qformer(audio_features, mask=None)

        if self.conv_ds is not None:
            if masks is not None:
                masks = masks.squeeze(1)

            audio_features, masks = self.conv_ds(audio_features, mask=masks)

        if self.linear_downsample_rate != 1:
            bs, seq_len, feat_dim = audio_features.size()
            padding = seq_len % self.linear_downsample_rate
            if padding > 0:
                audio_features = F.pad(
                    audio_features,
                    (0, 0, 0, self.linear_downsample_rate - padding),
                    "constant",
                    0,
                )

            seq_len = audio_features.size(1)
            audio_features = audio_features.view(
                bs,
                seq_len // self.linear_downsample_rate,
# ... truncated for brevity ...
```
**EN:** This method implements `get_audio_features(input_embeds: ..., audio_attention_mask: ...=..., audio_projection_mode: ...=...)` and arguments:.
**CN:** 这个方法实现了 `get_audio_features(input_embeds: ..., audio_attention_mask: ...=..., audio_projection_mode: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1242-1260: Method `AudioEmbedding.forward`
```python
    def forward(
        self,
        audio_features: torch.FloatTensor,
        audio_attention_mask: torch.Tensor = None,
        audio_projection_mode: str = "speech",
    ) -> torch.FloatTensor:
        """
        arguments:
            audio_features: audio features (num_audio_tokens, T, D)

        returns:
            audio_embeds: audio embeddings (num_audio_tokens, hidden_dim)
        """
        audio_embeds = self.get_audio_features(
            audio_features,
            audio_attention_mask=audio_attention_mask,
            audio_projection_mode=audio_projection_mode,
        )
        return audio_embeds
```
**EN:** This method implements `forward(audio_features: ..., audio_attention_mask: ...=..., audio_projection_mode: ...=...)` and arguments:.
**CN:** 这个方法实现了 `forward(audio_features: ..., audio_attention_mask: ...=..., audio_projection_mode: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影
- **EN:** Multimodal feature processing / **CN:** 多模态特征处理
- **EN:** Checkpoint remapping and weight loading / **CN:** 检查点重映射与权重加载

## Dependencies / 依赖关系
- `abc`
- `math`
- `typing.Literal`
- `typing.Optional`
- `numpy`
- `torch`
- `torch.nn.functional`
- `torch.Tensor`
- `torch.nn`
- `torch.distributed.algorithms._checkpoint.checkpoint_wrapper.CheckpointWrapper`
- `torch.distributed.fsdp.fully_sharded_data_parallel.FullyShardedDataParallel`
- `transformers.PretrainedConfig`
- `sglang.srt.models.phi4mm_utils.AbsolutePositionalEncoding`
- `sglang.srt.models.phi4mm_utils.ConvModule`
- `sglang.srt.models.phi4mm_utils.FeedForward`
- `sglang.srt.models.phi4mm_utils.MeanVarianceNormLayer`
- `sglang.srt.models.phi4mm_utils.MultiHeadedAttention`
- `sglang.srt.models.phi4mm_utils.MultiSequential`
- `sglang.srt.models.phi4mm_utils.NemoConvSubsampling`
- `sglang.srt.models.phi4mm_utils.T5RelativeAttentionLogitBias`
- `sglang.srt.models.phi4mm_utils.adaptive_enc_mask`
- `sglang.srt.models.phi4mm_utils.get_offset`
- `sglang.srt.models.phi4mm_utils.unfold_tensor`
