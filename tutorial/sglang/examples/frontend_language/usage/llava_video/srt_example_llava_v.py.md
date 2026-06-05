# srt_example_llava_v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/llava_video/srt_example_llava_v.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example runs LLaVA-Video inference over subtitle-timestamped clips and saves chunked results for later aggregation. / 该示例会基于字幕时间片段执行 LLaVA-Video 推理，并保存分块结果以便后续汇总。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module overview and usage
````python
"""
Usage:
pip install opencv-python-headless

python3 srt_example_llava_v.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 8-16: Import dependencies and runtime symbols
````python
import argparse
import csv
import json
import os
import time

import requests

import sglang as sgl
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 19-22: Define video_qa prompt program
````python
@sgl.function
def video_qa(s, num_frames, video_path, question):
    s += sgl.user(sgl.video(video_path, num_frames) + question)
    s += sgl.assistant(sgl.gen("answer"))
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 25-33: Run one request
````python
def single(path, num_frames=16):
    state = video_qa.run(
        num_frames=num_frames,
        video_path=path,
        question="Please provide a detailed description of the video, focusing on the main subjects, their actions, the background scenes",
        temperature=0.0,
        max_new_tokens=1024,
    )
    print(state["answer"], "\n")
````
**EN:** This function runs the example once with concrete inputs and prints the returned result in a human-readable form.
**CN:** 该函数会使用具体输入执行一次示例，并以便于阅读的形式打印返回结果。

### Lines 36-47: Split into chunks
````python
def split_into_chunks(lst, num_chunks):
    """Split a list into a specified number of chunks."""
    # Calculate the chunk size using integer division. Note that this may drop some items if not evenly divisible.
    chunk_size = len(lst) // num_chunks

    if chunk_size == 0:
        chunk_size = len(lst)
    # Use list comprehension to generate chunks. The last chunk will take any remainder if the list size isn't evenly divisible.
    chunks = [lst[i : i + chunk_size] for i in range(0, len(lst), chunk_size)]
    # Ensure we have exactly num_chunks chunks, even if some are empty
    chunks.extend([[] for _ in range(num_chunks - len(chunks))])
    return chunks
````
**EN:** This function encapsulates the “Split into chunks” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Split into chunks”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 50-57: Save batch results
````python
def save_batch_results(batch_video_files, states, cur_chunk, batch_idx, save_dir):
    csv_filename = f"{save_dir}/chunk_{cur_chunk}_batch_{batch_idx}.csv"
    with open(csv_filename, "w", newline="") as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow(["video_name", "answer"])
        for video_path, state in zip(batch_video_files, states):
            video_name = os.path.basename(video_path)
            writer.writerow([video_name, state["answer"]])
````
**EN:** This function encapsulates the “Save batch results” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Save batch results”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 60-72: Handle and cleanup final results
````python
def compile_and_cleanup_final_results(cur_chunk, num_batches, save_dir):
    final_csv_filename = f"{save_dir}/final_results_chunk_{cur_chunk}.csv"
    with open(final_csv_filename, "w", newline="") as final_csvfile:
        writer = csv.writer(final_csvfile)
        writer.writerow(["video_name", "answer"])
        for batch_idx in range(num_batches):
            batch_csv_filename = f"{save_dir}/chunk_{cur_chunk}_batch_{batch_idx}.csv"
            with open(batch_csv_filename, "r") as batch_csvfile:
                reader = csv.reader(batch_csvfile)
                next(reader)  # Skip header row
                for row in reader:
                    writer.writerow(row)
            os.remove(batch_csv_filename)
````
**EN:** This function encapsulates the “Handle and cleanup final results” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle and cleanup final results”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 75-87: Handle video files
````python
def find_video_files(video_dir):
    # Check if the video_dir is actually a file
    if os.path.isfile(video_dir):
        # If it's a file, return it as a single-element list
        return [video_dir]

    # Original logic to find video files in a directory
    video_files = []
    for root, dirs, files in os.walk(video_dir):
        for file in files:
            if file.endswith((".mp4", ".avi", ".mov")):
                video_files.append(os.path.join(root, file))
    return video_files
````
**EN:** This function encapsulates the “Handle video files” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle video files”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 90-123: Execute batched requests
````python
def batch(video_dir, save_dir, cur_chunk, num_chunks, num_frames=16, batch_size=64):
    video_files = find_video_files(video_dir)
    chunked_video_files = split_into_chunks(video_files, num_chunks)[cur_chunk]
    num_batches = 0

    for i in range(0, len(chunked_video_files), batch_size):
        batch_video_files = chunked_video_files[i : i + batch_size]
        print(f"Processing batch of {len(batch_video_files)} video(s)...")

        if not batch_video_files:
            print("No video files found in the specified directory.")
            return

        batch_input = [
            {
                "num_frames": num_frames,
                "video_path": video_path,
                "question": "Please provide a detailed description of the video, focusing on the main subjects, their actions, the background scenes.",
            }
            for video_path in batch_video_files
        ]

        start_time = time.perf_counter()
        states = video_qa.run_batch(batch_input, max_new_tokens=512, temperature=0.2)
        total_time = time.perf_counter() - start_time
        average_time = total_time / len(batch_video_files)
        print(
            f"Number of videos in batch: {len(batch_video_files)}. Average processing time per video: {average_time:.2f} seconds. Total time for this batch: {total_time:.2f} seconds"
        )

        save_batch_results(batch_video_files, states, cur_chunk, num_batches, save_dir)
        num_batches += 1

    compile_and_cleanup_final_results(cur_chunk, num_batches, save_dir)
````
**EN:** This function sends multiple requests together and iterates over the returned batch results.
**CN:** 该函数会把多个请求一起发送，并遍历返回的批量结果。

### Lines 126-253: Program entry point
````python
if __name__ == "__main__":

    url = "https://raw.githubusercontent.com/EvolvingLMMs-Lab/sglang/dev/onevision_local/assets/jobs.mp4"

    cache_dir = os.path.expanduser("~/.cache")
    file_path = os.path.join(cache_dir, "jobs.mp4")

    os.makedirs(cache_dir, exist_ok=True)

    response = requests.get(url)
    response.raise_for_status()  # Raise an exception for bad responses

    with open(file_path, "wb") as f:
        f.write(response.content)

    print(f"File downloaded and saved to: {file_path}")
    # Create the parser
    parser = argparse.ArgumentParser(
        description="Run video processing with specified port."
    )

    # Add an argument for the port
    parser.add_argument(
        "--port",
        type=int,
        default=30000,
        help="The master port for distributed serving.",
    )
    parser.add_argument(
        "--chunk-idx", type=int, default=0, help="The index of the chunk to process."
    )
    parser.add_argument(
        "--num-chunks", type=int, default=8, help="The number of chunks to process."
    )
    parser.add_argument(
        "--save-dir",
        type=str,
        default="./work_dirs/llava_video",
        help="The directory to save the processed video files.",
    )
    parser.add_argument(
        "--video-dir",
        type=str,
        default=os.path.expanduser("~/.cache/jobs.mp4"),
        help="The directory or path for the processed video files.",
    )
    parser.add_argument(
        "--model-path",
        type=str,
        default="lmms-lab/LLaVA-NeXT-Video-7B",
        help="The model path for the video processing.",
    )
    parser.add_argument(
        "--num-frames",
        type=int,
        default=16,
        help="The number of frames to process in each video.",
    )
    parser.add_argument("--mm_spatial_pool_stride", type=int, default=2)

    # Parse the arguments
    args = parser.parse_args()
    cur_port = args.port
    cur_chunk = args.chunk_idx
    num_chunks = args.num_chunks
    num_frames = args.num_frames

    if "34b" in args.model_path.lower():
        tokenizer_path = "liuhaotian/llava-v1.6-34b-tokenizer"
    elif "7b" in args.model_path.lower():
        tokenizer_path = "llava-hf/llava-1.5-7b-hf"
    else:
        print("Invalid model path. Please specify a valid model path.")
        exit()

    model_override_args = {}
    model_override_args["mm_spatial_pool_stride"] = args.mm_spatial_pool_stride
    model_override_args["architectures"] = ["LlavaVidForCausalLM"]
    model_override_args["num_frames"] = args.num_frames
    model_override_args["model_type"] = "llava"

    if "34b" in args.model_path.lower():
        model_override_args["image_token_index"] = 64002

    if args.num_frames == 32:
        model_override_args["rope_scaling"] = {"factor": 2.0, "rope_type": "linear"}
        model_override_args["max_sequence_length"] = 4096 * 2
        model_override_args["tokenizer_model_max_length"] = 4096 * 2
    elif args.num_frames < 32:
        pass
    else:
        print(
            "The maximum number of frames to process is 32. Please specify a valid number of frames."
        )
        exit()

    runtime = sgl.Runtime(
        model_path=args.model_path,  # "liuhaotian/llava-v1.6-vicuna-7b",
        tokenizer_path=tokenizer_path,
        port=cur_port,
        json_model_override_args=json.dumps(model_override_args),
        tp_size=1,
    )
    sgl.set_default_backend(runtime)
    print(f"chat template: {runtime.endpoint.chat_template.name}")

    # Run a single request
    print("\n========== single ==========\n")
    root = args.video_dir
    if os.path.isfile(root):
        video_files = [root]
    else:
        video_files = [
            os.path.join(root, f)
            for f in os.listdir(root)
            if f.endswith((".mp4", ".avi", ".mov"))
        ]  # Add more extensions if needed
    start_time = time.perf_counter()  # Start time for processing a single video
    for cur_video in video_files[:1]:
        print(cur_video)
        single(cur_video, num_frames)
    end_time = time.perf_counter()  # End time for processing a single video
    total_time = end_time - start_time
    average_time = total_time / len(
        video_files
    )  # Calculate the average processing time
    print(f"Average processing time per video: {average_time:.2f} seconds")
    runtime.shutdown()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。
- **Batch inference / 批量推理**: The workflow processes multiple requests in one pass. / 该流程会在一次执行中处理多个请求。
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。
- **Distributed coordination / 分布式协同**: Multiple workers synchronize state, data, or updates. / 多个工作进程会同步状态、数据或更新。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, csv, json, os, time
- **Third-party / 第三方**: requests
- **Project-specific / 项目相关**: sglang
