# mla_preprocess.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/attention/mla_preprocess.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for attention execution and masking inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的注意力执行与掩码处理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module setup and shared state / 模块设置与共享状态
```python
import re
from functools import lru_cache
from typing import TYPE_CHECKING, Optional

import torch
import torch.nn.functional as F

from sglang.srt.hardware_backend.npu.utils import npu_format_cast
from sglang.srt.utils import get_bool_env_var

if TYPE_CHECKING:
    from sglang.srt.layers.quantization.base_config import QuantizationConfig
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `re`, `functools`, `typing`, `torch`, `torch.nn.functional`, `sglang.srt.hardware_backend.npu.utils`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `re`, `functools`, `typing`, `torch`, `torch.nn.functional`, `sglang.srt.hardware_backend.npu.utils`。

### Lines 15-17: Function `is_mla_preprocess_enabled` / 函数 `is_mla_preprocess_enabled`
```python
@lru_cache(maxsize=1)
def is_mla_preprocess_enabled() -> bool:
    return get_bool_env_var("SGLANG_NPU_USE_MLAPO")
```
**EN:** This function implements `is_mla_preprocess_enabled`. It primarily calls `lru_cache`, `get_bool_env_var` to complete its work.
**CN:** 该函数实现了 `is_mla_preprocess_enabled`。 它主要通过调用 `lru_cache`, `get_bool_env_var` 来完成任务。

### Lines 20-27: Function `is_fia_nz` / 函数 `is_fia_nz`
```python
@lru_cache(maxsize=1)
def is_fia_nz() -> bool:
    is_fia_nz_ = get_bool_env_var("SGLANG_USE_FIA_NZ")
    if is_fia_nz_:
        assert (
            is_mla_preprocess_enabled()
        ), "SGLANG_USE_FIA_NZ must be enable with SGLANG_NPU_USE_MLAPO"
    return is_fia_nz_
```
**EN:** This function implements `is_fia_nz`. It primarily calls `lru_cache`, `get_bool_env_var`, `is_mla_preprocess_enabled` to complete its work. State updates are written into `is_fia_nz_`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `is_fia_nz`。 它主要通过调用 `lru_cache`, `get_bool_env_var`, `is_mla_preprocess_enabled` 来完成任务。 状态更新主要写入 `is_fia_nz_`。 实现中使用了条件分支。

### Lines 30-33: Function `round_up` / 函数 `round_up`
```python
def round_up(val: int, align: int) -> int:
    if align == 0:
        return 0
    return -(val // -align) * align
```
**EN:** This function implements `round_up`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `round_up`。 实现中使用了条件分支。

### Lines 36-52: Function `transdata` / 函数 `transdata`
```python
def transdata(nd_mat, block_size: tuple = (16, 16)):
    r = round_up(nd_mat.shape[0], block_size[0])
    c = round_up(nd_mat.shape[1], block_size[1])
    r_pad = r - nd_mat.shape[0]
    c_pad = c - nd_mat.shape[1]
    nd_mat = F.pad(nd_mat, ((0, r_pad, 0, c_pad)))
    nz_mat = torch.permute(
        torch.reshape(
            nd_mat,
            (r // block_size[0], block_size[0], c // block_size[1], block_size[1]),
        ),
        [2, 0, 1, 3],
    )
    nz_mat = torch.reshape(
        nz_mat, (nz_mat.shape[0], nz_mat.shape[1] * nz_mat.shape[2], nz_mat.shape[3])
    )
    return nz_mat
```
**EN:** This function implements `transdata`. It primarily calls `round_up`, `F.pad`, `torch.permute`, `torch.reshape` to complete its work. State updates are written into `r`, `c`, `r_pad`, `c_pad`, `nd_mat`, `nz_mat`.
**CN:** 该函数实现了 `transdata`。 它主要通过调用 `round_up`, `F.pad`, `torch.permute`, `torch.reshape` 来完成任务。 状态更新主要写入 `r`, `c`, `r_pad`, `c_pad`, `nd_mat`, `nz_mat`。

### Lines 55-60: Function `trans_rope_weight` / 函数 `trans_rope_weight`
```python
def trans_rope_weight(weight, rope_dim):
    weight_1 = weight[..., -rope_dim::2, :].contiguous()
    weight_2 = weight[..., -rope_dim + 1 :: 2, :].contiguous()
    weight[..., -rope_dim:, :] = torch.cat([weight_1, weight_2], dim=-2)

    return weight.contiguous()
```
**EN:** This function implements `trans_rope_weight`. It primarily calls `weight.contiguous`, `torch.cat` to complete its work. State updates are written into `weight_1`, `weight_2`, `weight`.
**CN:** 该函数实现了 `trans_rope_weight`。 它主要通过调用 `weight.contiguous`, `torch.cat` 来完成任务。 状态更新主要写入 `weight_1`, `weight_2`, `weight`。

### Lines 63-63: Class `NPUFusedMLAPreprocess` declaration / 类 `NPUFusedMLAPreprocess` 声明
```python
class NPUFusedMLAPreprocess(torch.nn.Module):
```
**EN:** This class establishes `NPUFusedMLAPreprocess` as the main container/coordinator for the surrounding logic. It inherits from `torch.nn.Module`. Its core interface includes methods such as `__init__`, `preprocess_weights`, `mlaprolog_preprocess_weight`, `get_sin_cos`, `get_kv_cache_and_cache_idx`, `forward_absorb_prepare_npu_rms_norm_cache`.
**CN:** 该类将 `NPUFusedMLAPreprocess` 定义为周边逻辑的主要封装体或协调者。 它继承自 `torch.nn.Module`。 其核心接口包括 `__init__`, `preprocess_weights`, `mlaprolog_preprocess_weight`, `get_sin_cos`, `get_kv_cache_and_cache_idx`, `forward_absorb_prepare_npu_rms_norm_cache` 等方法。

### Lines 64-100: Method `NPUFusedMLAPreprocess.__init__` / 方法 `NPUFusedMLAPreprocess.__init__`
```python
    def __init__(
        self,
        fused_qkv_a_proj_with_mqa,
        q_a_layernorm,
        kv_a_layernorm,
        q_b_proj,
        w_kc,
        rotary_emb,
        layer_id,
        num_local_heads,
        qk_nope_head_dim,
        qk_rope_head_dim,
        v_head_dim,
        quant_config: Optional["QuantizationConfig"] = None,
    ):
        super().__init__()
        self.qkv_a_proj = fused_qkv_a_proj_with_mqa
        self.q_a_layernorm = q_a_layernorm
        self.kv_a_layernorm = kv_a_layernorm
        self.q_b_proj = q_b_proj
        self.w_kc = w_kc.contiguous()
        self.rotary_emb = rotary_emb
        self.layer_id = layer_id
        self.quant_config = quant_config
        self.has_preprocess_weights = False
        self.dtype = None

        self.q_lora_rank = self.q_b_proj.input_size  # 1536
# ... omitted for brevity ...
        self.qk_rope_head_dim = qk_rope_head_dim  # 64
        self.qk_head_dim = qk_nope_head_dim + qk_rope_head_dim
        self.v_head_dim = v_head_dim
        self.q_b_proj_weight_scale = self.q_b_proj.weight_scale.view(1, -1).to(
            torch.float
        )
```
**EN:** This method implements `__init__` on `NPUFusedMLAPreprocess`. It primarily calls `super.__init__`, `w_kc.contiguous`, `self.q_b_proj.weight_scale.view.to`, `super`, `self.q_b_proj.weight_scale.view` to complete its work. State updates are written into `self.qkv_a_proj`, `self.q_a_layernorm`, `self.kv_a_layernorm`, `self.q_b_proj`, `self.w_kc`, `self.rotary_emb`.
**CN:** 该方法（属于 `NPUFusedMLAPreprocess`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `w_kc.contiguous`, `self.q_b_proj.weight_scale.view.to`, `super`, `self.q_b_proj.weight_scale.view` 来完成任务。 状态更新主要写入 `self.qkv_a_proj`, `self.q_a_layernorm`, `self.kv_a_layernorm`, `self.q_b_proj`, `self.w_kc`, `self.rotary_emb`。

### Lines 102-237: Method `NPUFusedMLAPreprocess.preprocess_weights` / 方法 `NPUFusedMLAPreprocess.preprocess_weights`
```python
    def preprocess_weights(self, hidden_states):
        self.dummy = torch.zeros(
            (hidden_states.shape[-1]),
            dtype=hidden_states.dtype,
            device=hidden_states.device,
        )
        self.qkv_a_proj_input_offset = self.qkv_a_proj.input_offset.to(dtype=torch.int8)
        self.q_b_proj_input_offset = self.q_b_proj.input_offset.to(dtype=torch.int8)

        # matmul_0 weight [7168, 2112]
        fused_qkv_a_proj_with_mqa_weight_q = self.qkv_a_proj.weight.data[
            :, : self.q_lora_rank
        ].clone()  # [7168, 1536]
        fused_qkv_a_proj_with_mqa_weight_kv = self.qkv_a_proj.weight.data[
            :, self.q_lora_rank :
        ].clone()  # [7168, 576]
        # rope fit
        fused_qkv_a_proj_with_mqa_weight_kv_t = (
            fused_qkv_a_proj_with_mqa_weight_kv.t().contiguous()
        )
        fused_qkv_a_proj_with_mqa_weight_kv_t = trans_rope_weight(
            fused_qkv_a_proj_with_mqa_weight_kv_t, self.qk_rope_head_dim
        )
        fused_qkv_a_proj_with_mqa_weight_kv = (
            fused_qkv_a_proj_with_mqa_weight_kv_t.t().contiguous()
        )
        # cat nz
        fused_qkv_a_proj_with_mqa_weight_new = torch.cat(
# ... omitted for brevity ...
        q_b_proj_quant_bias = trans_rope_weight(
            q_b_proj_quant_bias, self.qk_rope_head_dim
        )
        self.q_b_proj_quant_bias = q_b_proj_quant_bias.reshape(
            self.num_local_heads * (self.qk_nope_head_dim + self.qk_rope_head_dim)
        )
```
**EN:** This method implements `preprocess_weights` on `NPUFusedMLAPreprocess`. It primarily calls `torch.zeros`, `self.qkv_a_proj.input_offset.to`, `self.q_b_proj.input_offset.to`, `self.qkv_a_proj.weight.data.clone`, `fused_qkv_a_proj_with_mqa_weight_kv.t.contiguous`, `trans_rope_weight` to complete its work. State updates are written into `self.dummy`, `self.qkv_a_proj_input_offset`, `self.q_b_proj_input_offset`, `fused_qkv_a_proj_with_mqa_weight_q`, `fused_qkv_a_proj_with_mqa_weight_kv`, `fused_qkv_a_proj_with_mqa_weight_kv_t`.
**CN:** 该方法（属于 `NPUFusedMLAPreprocess`）实现了 `preprocess_weights`。 它主要通过调用 `torch.zeros`, `self.qkv_a_proj.input_offset.to`, `self.q_b_proj.input_offset.to`, `self.qkv_a_proj.weight.data.clone`, `fused_qkv_a_proj_with_mqa_weight_kv.t.contiguous`, `trans_rope_weight` 来完成任务。 状态更新主要写入 `self.dummy`, `self.qkv_a_proj_input_offset`, `self.q_b_proj_input_offset`, `fused_qkv_a_proj_with_mqa_weight_q`, `fused_qkv_a_proj_with_mqa_weight_kv`, `fused_qkv_a_proj_with_mqa_weight_kv_t`。

### Lines 239-246: Method `NPUFusedMLAPreprocess.mlaprolog_preprocess_weight` / 方法 `NPUFusedMLAPreprocess.mlaprolog_preprocess_weight`
```python
    def mlaprolog_preprocess_weight(self):
        self.qkv_a_proj.weight.data = self.qkv_a_proj.weight.data.transpose(0, 1)
        qkv_a_proj_weight_q = self.qkv_a_proj.weight.data[:, : self.q_lora_rank].clone()
        qkv_a_proj_weight_kv = self.qkv_a_proj.weight.data[
            :, self.q_lora_rank :
        ].clone()
        self.q_a_proj_weight = npu_format_cast(qkv_a_proj_weight_q)
        self.kv_a_proj_weight = npu_format_cast(qkv_a_proj_weight_kv)
```
**EN:** This method implements `mlaprolog_preprocess_weight` on `NPUFusedMLAPreprocess`. It primarily calls `self.qkv_a_proj.weight.data.transpose`, `self.qkv_a_proj.weight.data.clone`, `npu_format_cast` to complete its work. State updates are written into `self.qkv_a_proj.weight.data`, `qkv_a_proj_weight_q`, `qkv_a_proj_weight_kv`, `self.q_a_proj_weight`, `self.kv_a_proj_weight`.
**CN:** 该方法（属于 `NPUFusedMLAPreprocess`）实现了 `mlaprolog_preprocess_weight`。 它主要通过调用 `self.qkv_a_proj.weight.data.transpose`, `self.qkv_a_proj.weight.data.clone`, `npu_format_cast` 来完成任务。 状态更新主要写入 `self.qkv_a_proj.weight.data`, `qkv_a_proj_weight_q`, `qkv_a_proj_weight_kv`, `self.q_a_proj_weight`, `self.kv_a_proj_weight`。

### Lines 248-253: Method `NPUFusedMLAPreprocess.get_sin_cos` / 方法 `NPUFusedMLAPreprocess.get_sin_cos`
```python
    def get_sin_cos(self, positions):
        cos_sin = self.rotary_emb.cos_sin_cache[positions]
        cos, sin = cos_sin.chunk(2, dim=-1)
        cos = cos.repeat(1, 2)
        sin = sin.repeat(1, 2)
        return cos, sin
```
**EN:** This method implements `get_sin_cos` on `NPUFusedMLAPreprocess`. It primarily calls `cos_sin.chunk`, `cos.repeat`, `sin.repeat` to complete its work. State updates are written into `cos_sin`, `cos`, `sin`.
**CN:** 该方法（属于 `NPUFusedMLAPreprocess`）实现了 `get_sin_cos`。 它主要通过调用 `cos_sin.chunk`, `cos.repeat`, `sin.repeat` 来完成任务。 状态更新主要写入 `cos_sin`, `cos`, `sin`。

### Lines 255-258: Method `NPUFusedMLAPreprocess.get_kv_cache_and_cache_idx` / 方法 `NPUFusedMLAPreprocess.get_kv_cache_and_cache_idx`
```python
    def get_kv_cache_and_cache_idx(self, forward_batch):
        k_cache, v_cache = forward_batch.token_to_kv_pool.get_kv_buffer(self.layer_id)
        slot_mapping = forward_batch.out_cache_loc.to(dtype=torch.int32)
        return k_cache, v_cache, slot_mapping
```
**EN:** This method implements `get_kv_cache_and_cache_idx` on `NPUFusedMLAPreprocess`. It primarily calls `forward_batch.token_to_kv_pool.get_kv_buffer`, `forward_batch.out_cache_loc.to` to complete its work. State updates are written into `slot_mapping`.
**CN:** 该方法（属于 `NPUFusedMLAPreprocess`）实现了 `get_kv_cache_and_cache_idx`。 它主要通过调用 `forward_batch.token_to_kv_pool.get_kv_buffer`, `forward_batch.out_cache_loc.to` 来完成任务。 状态更新主要写入 `slot_mapping`。

### Lines 260-339: Method `NPUFusedMLAPreprocess.forward_absorb_prepare_npu_rms_norm_cache` / 方法 `NPUFusedMLAPreprocess.forward_absorb_prepare_npu_rms_norm_cache`
```python
    def forward_absorb_prepare_npu_rms_norm_cache(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch,
        zero_allocator,
    ):
        bsz, _ = hidden_states.view(-1, hidden_states.shape[-1]).shape
        self.dtype = hidden_states.dtype
        if self.layer_id == 0:
            self.cos, self.sin = self.get_sin_cos(positions)
            self.rotary_emb.cos_cached, self.rotary_emb.sin_cache = self.cos, self.sin
        else:
            self.cos, self.sin = self.rotary_emb.cos_cached, self.rotary_emb.sin_cache

        self.kvCache, self.kvCacheRope, self.slotmapping = (
            self.get_kv_cache_and_cache_idx(forward_batch)
        )

        if not self.has_preprocess_weights:
            self.has_preprocess_weights = True

        cos, sin = self.cos, self.sin

        if self.q_lora_rank is not None:
            fused_qkv_a_proj_out = self.qkv_a_proj(hidden_states)[0]
            q_lowrank, latent_cache = fused_qkv_a_proj_out.split(
                [self.q_lora_rank, self.kv_lora_rank + self.qk_rope_head_dim], dim=-1
# ... omitted for brevity ...
            self.kvCache,
            epsilon=self.kv_a_layernorm.variance_epsilon,
            cache_mode=cache_mode,
        )

        return (q_pe, k_rope, q_nope, k_nope, forward_batch, zero_allocator, positions)
```
**EN:** This method implements `forward_absorb_prepare_npu_rms_norm_cache` on `NPUFusedMLAPreprocess`. It primarily calls `self.get_kv_cache_and_cache_idx`, `torch.split`, `q_nope.view`, `torch.matmul.transpose`, `q_pe.view`, `cos.view` to complete its work. State updates are written into `self.dtype`, `q_nope`, `q_pe`, `cos`, `sin`, `latent_cache`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUFusedMLAPreprocess`）实现了 `forward_absorb_prepare_npu_rms_norm_cache`。 它主要通过调用 `self.get_kv_cache_and_cache_idx`, `torch.split`, `q_nope.view`, `torch.matmul.transpose`, `q_pe.view`, `cos.view` 来完成任务。 状态更新主要写入 `self.dtype`, `q_nope`, `q_pe`, `cos`, `sin`, `latent_cache`。 实现中使用了条件分支。

### Lines 341-425: Method `NPUFusedMLAPreprocess.forward_mlapo` / 方法 `NPUFusedMLAPreprocess.forward_mlapo`
```python
    def forward_mlapo(self, positions, hidden_states, forward_batch, zero_allocator):
        input_dtype = hidden_states.dtype
        if not self.has_preprocess_weights:
            self.preprocess_weights(hidden_states)
            self.has_preprocess_weights = True
            self.dtype = hidden_states.dtype

        if self.layer_id == 0:
            cos, sin = self.get_sin_cos(positions)
            self.rotary_emb.cos_cached, self.rotary_emb.sin_cache = cos, sin
        else:
            cos, sin = self.rotary_emb.cos_cached, self.rotary_emb.sin_cache

        k_cache, v_cache, slot_mapping = self.get_kv_cache_and_cache_idx(forward_batch)

        q_nope_out = torch.empty(
            (hidden_states.shape[0], self.w_kc.shape[0], k_cache.shape[-1]),
            dtype=input_dtype,
            device=hidden_states.device,
        )
        q_rope_out = torch.empty(
            (hidden_states.shape[0], self.w_kc.shape[0], v_cache.shape[-1]),
            dtype=input_dtype,
            device=hidden_states.device,
        )
        if is_fia_nz():
            kv_shape, kv_rope_shape = k_cache.shape, v_cache.shape
            num_blocks, block_size, num_heads, _ = kv_shape
# ... omitted for brevity ...
            q_nope_out,
            k_cache,
            forward_batch,
            zero_allocator,
            positions,
        )
```
**EN:** This method implements `forward_mlapo` on `NPUFusedMLAPreprocess`. It primarily calls `self.get_kv_cache_and_cache_idx`, `torch.empty`, `is_fia_nz`, `hasattr`, `torch.ops.npu.mla_preprocess`, `self.preprocess_weights` to complete its work. State updates are written into `input_dtype`, `q_nope_out`, `q_rope_out`, `self.has_preprocess_weights`, `self.dtype`, `k_cache`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUFusedMLAPreprocess`）实现了 `forward_mlapo`。 它主要通过调用 `self.get_kv_cache_and_cache_idx`, `torch.empty`, `is_fia_nz`, `hasattr`, `torch.ops.npu.mla_preprocess`, `self.preprocess_weights` 来完成任务。 状态更新主要写入 `input_dtype`, `q_nope_out`, `q_rope_out`, `self.has_preprocess_weights`, `self.dtype`, `k_cache`。 实现中使用了条件分支。

### Lines 427-466: Method `NPUFusedMLAPreprocess.forward_mlaprolog` / 方法 `NPUFusedMLAPreprocess.forward_mlaprolog`
```python
    def forward_mlaprolog(self, positions, hidden_states, forward_batch):
        if not self.has_preprocess_weights:
            self.mlaprolog_preprocess_weight()
            self.has_preprocess_weights = True
        self.cos, self.sin = self.get_sin_cos(positions)
        k_cache, v_cache, slot_mapping = self.get_kv_cache_and_cache_idx(forward_batch)
        mla_prolog_input_args = {
            "token_x": hidden_states,
            "weight_dq": self.q_a_proj_weight,
            "weight_uq_qr": self.q_b_proj.weight,
            "weight_uk": self.w_kc,
            "weight_dkv_kr": self.kv_a_proj_weight,
            "rmsnorm_gamma_cq": self.q_a_layernorm.weight,
            "rmsnorm_gamma_ckv": self.kv_a_layernorm.weight,
            "rope_sin": self.sin,
            "rope_cos": self.cos,
            "kv_cache": k_cache,
            "kr_cache": v_cache,
            "cache_index": slot_mapping.to(dtype=torch.int64),
            "dequant_scale_w_uq_qr": self.q_b_proj_weight_scale,
            "rmsnorm_epsilon_cq": self.q_a_layernorm.variance_epsilon,
            "rmsnorm_epsilon_ckv": self.kv_a_layernorm.variance_epsilon,
            "cache_mode": "PA_BSND",
            "query_norm_flag": True,
            "weight_quant_mode": 1,  # 0:no quant; 1:uq_qr: quant; 2: weight_dq,weight_uq_qr,weight_dkv_kr: quant
        }
        q_nope, q_pe, dequant_scale_q_nope, qr, dequant_q_norm = (
            torch.ops.custom.npu_mla_prolog_v3(**mla_prolog_input_args)
# ... omitted for brevity ...
            k_cache,
            qr,
            forward_batch,
            positions,
            dequant_q_norm,
        )
```
**EN:** This method implements `forward_mlaprolog` on `NPUFusedMLAPreprocess`. It primarily calls `self.get_sin_cos`, `self.get_kv_cache_and_cache_idx`, `torch.ops.custom.npu_mla_prolog_v3`, `dequant_q_norm.view`, `self.mlaprolog_preprocess_weight`, `slot_mapping.to` to complete its work. State updates are written into `mla_prolog_input_args`, `dequant_q_norm`, `self.has_preprocess_weights`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUFusedMLAPreprocess`）实现了 `forward_mlaprolog`。 它主要通过调用 `self.get_sin_cos`, `self.get_kv_cache_and_cache_idx`, `torch.ops.custom.npu_mla_prolog_v3`, `dequant_q_norm.view`, `self.mlaprolog_preprocess_weight`, `slot_mapping.to` 来完成任务。 状态更新主要写入 `mla_prolog_input_args`, `dequant_q_norm`, `self.has_preprocess_weights`。 实现中使用了条件分支。

### Lines 468-489: Method `NPUFusedMLAPreprocess.forward` / 方法 `NPUFusedMLAPreprocess.forward`
```python
    def forward(self, positions, hidden_states, forward_batch, zero_allocator):
        # assert self.quant_config and self.quant_config.get_name() == "modelslim"
        # route by `qkv_a_proj` quant type as MTP layers can be unquantized
        _is_w8a8 = (
            hasattr(self.qkv_a_proj.quant_method, "quantization_config")
            and self.qkv_a_proj.quant_method.quantization_config.get_name()
            == "modelslim"
        )
        # with the mlaprolog enabled, the kv_b_proj layers are unquantized
        _is_mlaprolog = hasattr(self.quant_config, "ignore") and any(
            re.fullmatch(r".*kv_b_proj", l) for l in self.quant_config.ignore
        )
        if _is_w8a8:
            return self.forward_mlapo(
                positions, hidden_states, forward_batch, zero_allocator
            )
        elif _is_mlaprolog:
            return self.forward_mlaprolog(positions, hidden_states, forward_batch)
        else:
            return self.forward_absorb_prepare_npu_rms_norm_cache(
                positions, hidden_states, forward_batch, zero_allocator
            )
```
**EN:** This method implements `forward` on `NPUFusedMLAPreprocess`. It primarily calls `hasattr`, `any`, `self.forward_mlapo`, `self.qkv_a_proj.quant_method.quantization_config.get_name`, `self.forward_mlaprolog`, `self.forward_absorb_prepare_npu_rms_norm_cache` to complete its work. State updates are written into `_is_w8a8`, `_is_mlaprolog`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUFusedMLAPreprocess`）实现了 `forward`。 它主要通过调用 `hasattr`, `any`, `self.forward_mlapo`, `self.qkv_a_proj.quant_method.quantization_config.get_name`, `self.forward_mlaprolog`, `self.forward_absorb_prepare_npu_rms_norm_cache` 来完成任务。 状态更新主要写入 `_is_w8a8`, `_is_mlaprolog`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `NPUFusedMLAPreprocess`
- **Functions / 函数**: `is_mla_preprocess_enabled`, `is_fia_nz`, `round_up`, `transdata`, `trans_rope_weight`, `__init__`, `preprocess_weights`, `mlaprolog_preprocess_weight`
- **Themes / 主题**: `attention`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.npu.utils`, `sglang.srt.utils`, `sglang.srt.layers.quantization.base_config`
- **External / 外部依赖**: `torch`, `torch.nn.functional`
- **Standard library / 标准库**: `re`, `functools`, `typing`
