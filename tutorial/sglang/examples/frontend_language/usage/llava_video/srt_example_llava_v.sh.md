# srt_example_llava_v.sh — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/llava_video/srt_example_llava_v.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script partitions a video inference workload across nodes and GPUs, repeatedly launching the paired Python worker until each chunk succeeds. / 该 Shell 脚本会将视频推理任务划分到多个节点和 GPU 上，并反复启动配套 Python 工作进程直到每个分块成功完成。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Document distributed usage
````bash
#!/bin/bash

##### USAGE #####
#    - First node:
#      ```sh
#      bash examples/usage/llava_video/srt_example_llava_v.sh K 0 YOUR_VIDEO_PATH YOUR_MODEL_PATH FRAMES_PER_VIDEO
#      ```
#    - Second node:
#      ```sh
#      bash examples/usage/llava_video/srt_example_llava_v.sh K 1 YOUR_VIDEO_PATH YOUR_MODEL_PATH FRAMES_PER_VIDEO
#      ```
#    - The K node:
#      ```sh
#      bash examples/usage/llava_video/srt_example_llava_v.sh K K-1 YOUR_VIDEO_PATH YOUR_MODEL_PATH FRAMES_PER_VIDEO
#      ```
````
**EN:** This block explains how each node should invoke the script and which positional arguments must be provided.
**CN:** 该代码块说明了每个节点应如何调用脚本，以及必须提供哪些位置参数。

### Lines 18-28: Resolve script location and environment
````bash
# Replace `K`, `YOUR_VIDEO_PATH`, `YOUR_MODEL_PATH`, and `FRAMES_PER_VIDEO` with your specific details.
# CURRENT_ROOT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
CURRENT_ROOT=$(dirname "$0")

echo ${CURRENT_ROOT}

cd ${CURRENT_ROOT}

export PYTHONWARNINGS=ignore

START_TIME=$(date +%s)  # Capture start time
````
**EN:** The script discovers its own directory, switches into it, and sets shell environment variables before launching work.
**CN:** 脚本会定位自身目录、切换到该目录，并在启动任务前设置所需的 Shell 环境变量。

### Lines 30-38: Parse positional arguments
````bash
NUM_NODES=$1

CUR_NODES_IDX=$2

VIDEO_DIR=$3

MODEL_PATH=$4

NUM_FRAMES=$5
````
**EN:** These assignments map command-line arguments to readable variable names used throughout the orchestration logic.
**CN:** 这些赋值语句把命令行参数映射成更易读的变量名，供后续编排逻辑使用。

### Lines 41-57: Shell orchestration step
````bash
# FRAME_FORMAT=$6

# FRAME_FORMAT=$(echo $FRAME_FORMAT | tr '[:lower:]' '[:upper:]')

# # Check if FRAME_FORMAT is either JPEG or PNG
# if [[ "$FRAME_FORMAT" != "JPEG" && "$FRAME_FORMAT" != "PNG" ]]; then
#     echo "Error: FRAME_FORMAT must be either JPEG or PNG."
#     exit 1
# fi

# export TARGET_FRAMES=$TARGET_FRAMES

echo "Each video you will sample $NUM_FRAMES frames"

# export FRAME_FORMAT=$FRAME_FORMAT

# echo "The frame format is $FRAME_FORMAT"
````
**EN:** This block contains one stage of the shell-based orchestration workflow.
**CN:** 该代码块表示基于 Shell 的编排流程中的一个阶段。

### Lines 59-70: Compute GPU partitioning
````bash
# Assuming GPULIST is a bash array containing your GPUs
GPULIST=(0 1 2 3 4 5 6 7)
LOCAL_CHUNKS=${#GPULIST[@]}

echo "Number of GPUs in GPULIST: $LOCAL_CHUNKS"

ALL_CHUNKS=$((NUM_NODES * LOCAL_CHUNKS))

# Calculate GPUs per chunk
GPUS_PER_CHUNK=1

echo $GPUS_PER_CHUNK
````
**EN:** This block defines the local GPU list and derives how the full workload should be split across nodes and devices.
**CN:** 该代码块定义了本地 GPU 列表，并据此推导整个任务如何在多个节点和设备之间切分。

### Lines 72-123: Launch chunk workers with retries
````bash
for IDX in $(seq 1 $LOCAL_CHUNKS); do
    (
        START=$(((IDX-1) * GPUS_PER_CHUNK))
        LENGTH=$GPUS_PER_CHUNK # Length for slicing, not the end index

        CHUNK_GPUS=(${GPULIST[@]:$START:$LENGTH})

        # Convert the chunk GPUs array to a comma-separated string
        CHUNK_GPUS_STR=$(IFS=,; echo "${CHUNK_GPUS[*]}")

        LOCAL_IDX=$((CUR_NODES_IDX * LOCAL_CHUNKS + IDX))

        echo "Chunk $(($LOCAL_IDX - 1)) will run on GPUs $CHUNK_GPUS_STR"

        # Calculate the port for this chunk. Ensure it's incremented by 5 for each chunk.
        PORT=$((10000 + RANDOM % 55536))

        MAX_RETRIES=10
        RETRY_COUNT=0
        COMMAND_STATUS=1  # Initialize as failed

        while [ $RETRY_COUNT -lt $MAX_RETRIES ] && [ $COMMAND_STATUS -ne 0 ]; do
            echo "Running chunk $(($LOCAL_IDX - 1)) on GPUs $CHUNK_GPUS_STR with port $PORT. Attempt $(($RETRY_COUNT + 1))"

#!/bin/bash
            CUDA_VISIBLE_DEVICES=$CHUNK_GPUS_STR python3 srt_example_llava_v.py \
            --port $PORT \
            --num-chunks $ALL_CHUNKS \
            --chunk-idx $(($LOCAL_IDX - 1)) \
            --save-dir work_dirs/llava_next_video_inference_results \
            --video-dir $VIDEO_DIR \
            --model-path $MODEL_PATH \
            --num-frames $NUM_FRAMES #&

            wait $!  # Wait for the process to finish and capture its exit status
            COMMAND_STATUS=$?

            if [ $COMMAND_STATUS -ne 0 ]; then
                echo "Execution failed for chunk $(($LOCAL_IDX - 1)), attempt $(($RETRY_COUNT + 1)). Retrying..."
                RETRY_COUNT=$(($RETRY_COUNT + 1))
                sleep 180  # Wait a bit before retrying
            else
                echo "Execution succeeded for chunk $(($LOCAL_IDX - 1))."
            fi
        done

        if [ $COMMAND_STATUS -ne 0 ]; then
            echo "Execution failed for chunk $(($LOCAL_IDX - 1)) after $MAX_RETRIES attempts."
        fi
    ) #&
    sleep 2  # Slight delay to stagger the start times
done
````
**EN:** The loop starts one Python worker per chunk, pins it to a GPU subset, and retries failed executions until the retry budget is exhausted.
**CN:** 该循环会为每个分块启动一个 Python 工作进程，将其绑定到指定 GPU 子集，并在重试次数耗尽前持续重试失败任务。

### Lines 125-131: Merge results and report timing
````bash
wait

cat work_dirs/llava_next_video_inference_results/final_results_chunk_*.csv > work_dirs/llava_next_video_inference_results/final_results_node_${CUR_NODES_IDX}.csv

END_TIME=$(date +%s)  # Capture end time
ELAPSED_TIME=$(($END_TIME - $START_TIME))
echo "Total execution time: $ELAPSED_TIME seconds."
````
**EN:** After all workers finish, the script concatenates per-chunk CSV outputs and prints the total elapsed runtime.
**CN:** 所有工作进程结束后，脚本会拼接各分块的 CSV 输出，并打印总耗时。

## Key Concepts / 关键概念
- **Distributed coordination / 分布式协同**: Multiple workers synchronize state, data, or updates. / 多个工作进程会同步状态、数据或更新。
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。
- **Batch inference / 批量推理**: The workflow processes multiple requests in one pass. / 该流程会在一次执行中处理多个请求。

## Dependencies / 依赖关系
- **Shell tools / Shell 工具**: bash, dirname, date, seq, python3, cat, sleep, wait, CUDA runtime, srt_example_llava_v.py
