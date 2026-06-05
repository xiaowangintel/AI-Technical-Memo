# main.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/apps/webui/main.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `add_webui_args`, and `run_sgl_diffusion_webui`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于多模态生成模块。它围绕 `add_webui_args` 和 `run_sgl_diffusion_webui` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module setup and imports / 模块初始化与导入
```python
import argparse
import os

from sglang.multimodal_gen.configs.sample.sampling_params import (
    DataType,
    SamplingParams,
)
from sglang.multimodal_gen.runtime.entrypoints.utils import (
    post_process_sample,
    prepare_request,
)
from sglang.multimodal_gen.runtime.scheduler_client import sync_scheduler_client
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.srt.environ import envs
```
**EN:** This block establishes the module context and imports `argparse`, `os`, `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.scheduler_client`, and `sglang.multimodal_gen.runtime.server_args`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `argparse`、`os`、`sglang.multimodal_gen.configs.sample.sampling_params`、`sglang.multimodal_gen.runtime.entrypoints.utils`、`sglang.multimodal_gen.runtime.scheduler_client` 和 `sglang.multimodal_gen.runtime.server_args`。这些依赖为后续实现提供所需符号。

### Lines 17-17: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 20-24: `add_webui_args` implementation / `add_webui_args` 实现
```python
def add_webui_args(parser: argparse.ArgumentParser):
    """Add the arguments for the generate command."""
    parser = ServerArgs.add_cli_args(parser)
    parser = SamplingParams.add_cli_args(parser)
    return parser
```
**EN:** This block defines function `add_webui_args`. Add the arguments for the generate command. Key calls include `ServerArgs.add_cli_args`, and `SamplingParams.add_cli_args`. Parameters such as `parser` drive the behavior in this section.
**CN:** 该代码块定义了函数 `add_webui_args`。 它用于处理 add webui args 相关逻辑。 关键调用包括 `ServerArgs.add_cli_args` 和 `SamplingParams.add_cli_args`。 本段逻辑主要由 `parser` 等参数驱动。

### Lines 27-253: `run_sgl_diffusion_webui` implementation / `run_sgl_diffusion_webui` 实现
```python
def run_sgl_diffusion_webui(server_args: ServerArgs):
    # import gradio in function to avoid CI crash

    import gradio as gr

    def resolve_model_repo_id(model_path: str) -> str:
        from pathlib import Path

        from huggingface_hub.utils import HFValidationError, validate_repo_id

        try:
            validate_repo_id(model_path)
            return model_path
        except HFValidationError:
            pass

        p = Path(model_path).expanduser()
        parts = p.parts

        if len(parts) < 2:
            raise ValueError(f"Invalid model_path: {model_path}")

        candidate = f"{parts[-2]}/{parts[-1]}"
        validate_repo_id(candidate)  # let it raise if invalid
        return candidate

    repo_id = resolve_model_repo_id(server_args.model_path)
    if envs.SGLANG_USE_MODELSCOPE.get():
        from modelscope.hub.api import HubApi

        api = HubApi()
        model_info_obj = api.model_info(repo_id)
        task_name = model_info_obj.tasks[0]["Name"].replace("-synthesis", "")
    else:
        from huggingface_hub import model_info

        task_name = model_info(repo_id).pipeline_tag

    # init client
    sync_scheduler_client.initialize(server_args)

    if task_name in ("text-to-video", "image-to-video", "video-to-video"):
        task_type = "video"
    elif task_name in ["text-to-image", "image-to-image"]:
        task_type = "image"
    else:
        raise ValueError(
            f"The task name {task_name} of model {server_args.model_path} is not a valid task name. Please check the model path."
        )
    video_visible_only = task_type == "video"
    image_visible_only = task_type == "image"

    # server_args will be reused in gradio_generate function
    def gradio_generate(
        prompt,
        negative_prompt,
        reference_image_paths_str,
        seed,
        num_frames,
        frames_per_second,
        width,
        height,
        num_inference_steps,
        guidance_scale,
        enable_teacache,
    ):
        """
        NOTE: The input and output of function which is called by gradio button must be gradio components
        So we use global variable sampling_params_kwargs to avoid pass this param, because gradio does not support this.
        return [ np.ndarray, None ] | [None, np.ndarray]
        """
        if reference_image_paths_str:
            if "，" in reference_image_paths_str:
                logger.warning(
                    f"Warning: please use English comma to separate the reference image paths, and the reference image paths is: {reference_image_paths_str}"
                )
                reference_image_paths_str = reference_image_paths_str.replace("，", ",")
            image_path = [path.strip() for path in reference_image_paths_str.split(",")]
        else:
            image_path = None

        sampling_params_kwargs = dict(
            prompt=prompt,
            negative_prompt=negative_prompt,
            image_path=image_path,
            seed=seed,
            num_frames=num_frames,
            fps=frames_per_second,
            width=width,
            height=height,
            guidance_scale=guidance_scale,
            num_inference_steps=num_inference_steps,
            enable_teacache=enable_teacache,
            return_file_paths_only=False,
        )
        sampling_params = SamplingParams.from_user_sampling_params_args(
            server_args.model_path,
            server_args=server_args,
            **sampling_params_kwargs,
        )
        batch = prepare_request(
            server_args=server_args,
            sampling_params=sampling_params,
        )
        result = sync_scheduler_client.forward([batch])
        save_file_path = str(os.path.join(batch.output_path, batch.output_file_name))
        if result.output is None:
            sampling_params_str = "\n".join(
                [f"{key}: {value}" for key, value in sampling_params_kwargs.items()]
            )
            no_output_msg = f"No output is generated by client, and their sampling params is: {sampling_params_str}"

            if batch.data_type == DataType.VIDEO:
                if os.path.exists(save_file_path):
                    logger.warning(no_output_msg)
                    return None, save_file_path
                else:
                    no_output_msg += f"\nAnd the expected output file was not found at: {save_file_path}"
                    raise ValueError(no_output_msg)
            else:
                raise ValueError(no_output_msg)

        frames = post_process_sample(
            result.output[0],
            batch.data_type,
            batch.fps,
            batch.save_output,
            save_file_path,
        )
        if batch.data_type == DataType.VIDEO:
            # gradio video need video path to show video
            return None, save_file_path
        else:
            return frames[0], None

    with gr.Blocks() as demo:
        gr.Markdown("# 🚀 SGLang Diffusion Application")
        with gr.Row():
            launched_model_box = gr.Textbox(label="Model", value=server_args.model_path)
            task_name_box = gr.Textbox(label="Task name", value=task_name)

        with gr.Row():
            with gr.Column(scale=4):
                prompt = gr.Textbox(label="Prompt", value="A curious raccoon")
                negative_prompt = gr.Textbox(
                    label="Negative_prompt",
                    value="Bright tones, overexposed, static, blurred details, subtitles, style, works, paintings, images, static, overall gray, worst quality, low quality, JPEG compression residue, ugly, incomplete, extra fingers, poorly drawn hands, poorly drawn faces, deformed, disfigured, misshapen limbs, fused fingers, still picture, messy background, three legs, many people in the background, walking backwards",
                )
            with gr.Column(scale=1):
                seed = gr.Number(label="seed", precision=0, value=1234)
                run_btn = gr.Button("Generate", variant="primary", size="lg")

        with gr.Row():
            with gr.Column():
                width = gr.Number(label="width", precision=0, value=720)
                height = gr.Number(label="height", precision=0, value=480)
                num_inference_steps = gr.Slider(
                    minimum=0, maximum=50, value=20, step=1, label="num_inference_steps"
                )
                guidance_scale = gr.Slider(
                    minimum=0.0, maximum=10, value=5, step=0.01, label="guidance_scale"
                )
                num_frames = gr.Slider(
                    minimum=1,
                    maximum=181,
                    value=81,
                    step=1,
                    label="num_frames",
                    visible=video_visible_only,
                )
                frames_per_second = gr.Slider(
                    minimum=4,
                    maximum=60,
                    value=16,
                    step=1,
                    label="frames_per_second",
                    visible=video_visible_only,
                )
                reference_image_paths_str = gr.Textbox(
                    label="reference images",
                    placeholder="Examples: 'image1.png, image2.png' or 'https://example.com/image1.png, https://example.com/image2.png'",
                )
                enable_teacache = gr.Checkbox(label="enable_teacache", value=False)

            with gr.Column():
                image_out = gr.Image(
                    label="Generated Image", visible=image_visible_only, format="png"
                )
                video_out = gr.Video(
                    label="Generated Video", visible=video_visible_only
                )

        run_btn.click(
            fn=gradio_generate,
            inputs=[
                prompt,
                negative_prompt,
                reference_image_paths_str,
                seed,
                num_frames,
                frames_per_second,
                width,
                height,
                num_inference_steps,
                guidance_scale,
                enable_teacache,
            ],
            outputs=[image_out, video_out],
        )

        _, local_url, _ = demo.launch(
            server_port=server_args.webui_port,
            quiet=True,
            prevent_thread_lock=True,
            show_error=True,
        )

        # print banner
        delimiter = "=" * 80
        url = local_url or f"http://localhost:{server_args.webui_port}"
        print(f"""
{delimiter}
\033[1mSGLang Diffusion WebUI available at:\033[0m \033[1;4;92m{url}\033[0m
{delimiter}
""")

        demo.block_thread()
```
**EN:** This block defines function `run_sgl_diffusion_webui`. It runs sgl diffusion webui. Key calls include `resolve_model_repo_id`, `envs.SGLANG_USE_MODELSCOPE.get`, `sync_scheduler_client.initialize`, `Path.expanduser`, and `validate_repo_id`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `server_args` drive the behavior in this section.
**CN:** 该代码块定义了函数 `run_sgl_diffusion_webui`。 它用于运行sgl diffusion webui。 关键调用包括 `resolve_model_repo_id`、`envs.SGLANG_USE_MODELSCOPE.get`、`sync_scheduler_client.initialize`、`Path.expanduser` 和 `validate_repo_id`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `server_args` 等参数驱动。

## Key Concepts / 关键概念
- `add_webui_args`: Add the arguments for the generate command. / 顶层函数，用于处理 add webui args 相关逻辑。
- `run_sgl_diffusion_webui`: Top-level function that runs sgl diffusion webui. / 顶层函数，用于运行sgl diffusion webui。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `os`, `pathlib`
- **Third-party / 第三方依赖**: `gradio`, `huggingface_hub.utils`, `modelscope.hub.api`, `huggingface_hub`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.scheduler_client`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.environ`

- **Total lines / 总行数**: 253
