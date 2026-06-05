# reward_model.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/reward_model.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This runtime example scores candidates instead of generating long-form text, enabling ranking-oriented workflows. / 该运行时示例返回的是候选项分数而不是长文本，因此适合排序类工作流。

## Line-by-Line Analysis / 逐行分析
### Lines 4-4: Import dependencies and runtime symbols
````python
import requests
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 6-32: Set top-level configuration
````python
url = "http://127.0.0.1:30000"

PROMPT = (
    "What is the range of the numeric output of a sigmoid node in a neural network?"
)
RESPONSE1 = "The output of a sigmoid node is bounded between -1 and 1."
RESPONSE2 = "The output of a sigmoid node is bounded between 0 and 1."

json_data = {
    "conv": [
        [
            {"role": "user", "content": PROMPT},
            {"role": "assistant", "content": RESPONSE1},
        ],
        [
            {"role": "user", "content": PROMPT},
            {"role": "assistant", "content": RESPONSE2},
        ],
    ],
}
response = requests.post(
    url + "/classify",
    json=json_data,
).json()

print(response)
print("scores:", [x["embedding"] for x in response])
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

## Key Concepts / 关键概念
- **Scoring and reranking / 打分与重排序**: The model returns scores used for ranking candidates. / 模型返回可用于候选项排序的分数。

## Dependencies / 依赖关系
- **Third-party / 第三方**: requests
