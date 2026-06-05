# training_loss.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/training_loss.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import argparse
 2: import inspect
 3: import os
 4: import sys
 5: import time
 6: from datetime import timedelta
 7: 
 8: from datasets import load_dataset, load_metric
 9: from transformers import AutoModelForSequenceClassification, AutoTokenizer
10: 
11: import torch
12: import torch._dynamo
13: from torch.utils.data import DataLoader
14: 
15: 
16: torch.backends.cuda.matmul.allow_tf32 = True
17: 
18: # You will download around 84G dataset if you run this end to end training/evaluation example.
19: 
20: os.environ["TOKENIZERS_PARALLELISM"] = "false"
21: device = torch.device("cuda") if torch.cuda.is_available() else torch.device("cpu")
22: 
23: 
24: def data_processing(num_samples, batch_size):
25:     dataset = load_dataset("yelp_review_full")
26:     tokenizer = AutoTokenizer.from_pretrained("bert-base-cased")
27: 
28:     def tokenize_function(examples):
29:         return tokenizer(examples["text"], padding="max_length", truncation=True)
30: 
31:     tokenized_datasets = dataset.map(tokenize_function, batched=True)
32: 
33:     tokenized_datasets = tokenized_datasets.remove_columns(["text"])
34:     tokenized_datasets = tokenized_datasets.rename_column("label", "labels")
35:     tokenized_datasets.set_format("torch")
36: 
37:     small_train_dataset = tokenized_datasets["train"].select(range(num_samples))
38:     small_eval_dataset = tokenized_datasets["test"].select(range(num_samples))
39: 
40:     train_dataloader = DataLoader(small_train_dataset, batch_size=batch_size)
````
- EN: Handles module imports such as `argparse`, `inspect`, `os`, `sys`.
- CN: 处理模块导入，例如 `argparse`, `inspect`, `os`, `sys`。
- EN: Implements callable logic such as `data_processing`, `tokenize_function`.
- CN: 实现可调用逻辑，例如 `data_processing`, `tokenize_function`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````python
41:     eval_dataloader = DataLoader(small_eval_dataset, batch_size=batch_size)
42: 
43:     return train_dataloader, eval_dataloader
44: 
45: 
46: def training_iter_fn(batch, model, optimizer):
47:     outputs = model(**batch)
48:     loss = outputs.loss
49:     loss.backward()
50:     optimizer.step()
51:     optimizer.zero_grad()
52:     return loss
53: 
54: 
55: def model_training_evaluation(
56:     backend, train_dataloader, eval_dataloader, model, optimizer, num_epochs, evaluation
57: ):
58:     model.to(device)
59:     model.train()
60:     loss_history = []
61:     if not backend:
62:         # Run with native Pytorch
63:         opt_training_iter_fn = training_iter_fn
64:     else:
65:         # Support backends: eager, aot_eager, aot_nvfuser and inductor
66:         opt_training_iter_fn = torch._dynamo.optimize(backend)(training_iter_fn)
67:     for epoch in range(num_epochs):
68:         running_loss = 0.0
69:         for i, batch in enumerate(train_dataloader, 0):
70:             batch = {k: v.to(device) for k, v in batch.items()}
71:             loss = opt_training_iter_fn(batch, model, optimizer)
72:             running_loss += loss.item()
73:             if i % 100 == 99:
74:                 loss_history.append(running_loss / 100)
75:                 running_loss = 0.0
76: 
77:     if evaluation:
78:         metric = load_metric("accuracy")
79:         model.eval()
80:         if not backend:
````
- EN: Implements callable logic such as `training_iter_fn`, `model_training_evaluation`.
- CN: 实现可调用逻辑，例如 `training_iter_fn`, `model_training_evaluation`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81:             opt_model = model
 82:         else:
 83:             opt_model = torch._dynamo.optimize(backend)(model)
 84:         for batch in eval_dataloader:
 85:             batch = {k: v.to(device) for k, v in batch.items()}
 86:             with torch.no_grad():
 87:                 outputs = opt_model(**batch)
 88: 
 89:             logits = outputs.logits
 90:             predictions = torch.argmax(logits, dim=-1)
 91:             metric.add_batch(predictions=predictions, references=batch["labels"])
 92: 
 93:         return loss_history, metric.compute()
 94:     else:
 95:         return loss_history, None
 96: 
 97: 
 98: def check_loss(ref_loss, res_loss):
 99:     if len(ref_loss) != len(res_loss):
100:         raise AssertionError(
101:             f"Expected loss lists to have equal length, but got {len(ref_loss)} and {len(res_loss)}"
102:         )
103:     length = len(ref_loss)
104:     x = min(length, 10)
105:     return sum(res_loss[-x:]) / 10 <= sum(ref_loss[-x:]) / 10 + 0.1
106: 
107: 
108: def parse_args():
109:     parser = argparse.ArgumentParser(
110:         description="TorchDynamo end to end training/evaluation benchmark"
111:     )
112:     parser.add_argument(
113:         "--epochs", type=int, default=10, help="number of epochs to train (default: 10)"
114:     )
115:     parser.add_argument(
116:         "--num-samples",
117:         type=int,
118:         default=1000,
119:         help="number of samples to train/eval (default: 1000)",
120:     )
````
- EN: Implements callable logic such as `check_loss`, `parse_args`.
- CN: 实现可调用逻辑，例如 `check_loss`, `parse_args`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````python
121:     parser.add_argument(
122:         "--batch-size",
123:         type=int,
124:         default=8,
125:         help="input batch size for training (default: 8)",
126:     )
127:     parser.add_argument(
128:         "--lr", type=float, default=5e-5, help="learning rate (default: 5e-5)"
129:     )
130:     parser.add_argument(
131:         "--backend",
132:         choices=torch._dynamo.list_backends(exclude_tags=None),
133:         default="inductor",
134:         help="train/evaluate model with a given backend (default: inductor)",
135:     )
136:     parser.add_argument(
137:         "--optimizer",
138:         default="Adam",
139:         help="train model using a given optimizer (default: Adam)",
140:     )
141:     parser.add_argument(
142:         "--evaluation",
143:         action="store_true",
144:         help="running evaluation after model training",
145:     )
146:     args = parser.parse_args()
147:     return args
148: 
149: 
150: def main():
151:     args = parse_args()
152:     train_dataloader, eval_dataloader = data_processing(
153:         args.num_samples, args.batch_size
154:     )
155:     model = AutoModelForSequenceClassification.from_pretrained(
156:         "bert-base-cased", num_labels=5
157:     )
158:     optimizer_cls = getattr(sys.modules["torch.optim"], args.optimizer)
159:     if "capturable" in inspect.signature(optimizer_cls).parameters:
160:         optimizer = optimizer_cls(model.parameters(), lr=args.lr, capturable=True)
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-200
````python
161:     else:
162:         optimizer = optimizer_cls(model.parameters(), lr=args.lr)
163:     native_start = time.time()
164:     ref_loss, accuracy = model_training_evaluation(
165:         None,
166:         train_dataloader,
167:         eval_dataloader,
168:         model,
169:         optimizer,
170:         args.epochs,
171:         args.evaluation,
172:     )
173:     native_end = time.time()
174:     res_loss, accuracy = model_training_evaluation(
175:         args.backend,
176:         train_dataloader,
177:         eval_dataloader,
178:         model,
179:         optimizer,
180:         args.epochs,
181:         args.evaluation,
182:     )
183:     dynamo_end = time.time()
184:     if check_loss(ref_loss, res_loss):
185:         print(
186:             "[PASSED] TorchDynamo end to end training loss is less than or equal to native PyTorch"
187:         )
188:     else:
189:         print(
190:             "[FAILED] TorchDynamo end to end training loss is greater than native Pytorch"
191:         )
192:     if args.evaluation:
193:         print(f"Model accuracy: {accuracy}")
194:     native_elapsed = native_end - native_start
195:     dynamo_elapsed = dynamo_end - native_end
196:     print(
197:         f"Train model on {args.epochs} epochs with backend {args.backend} and optimizer {args.optimizer}:"
198:     )
199:     print(f"PyTorch spent {timedelta(seconds=native_elapsed / args.epochs)} per epoch")
200:     print(
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 201-206
````python
201:         f"TorchDynamo spent {timedelta(seconds=dynamo_elapsed / args.epochs)} per epoch"
202:     )
203: 
204: 
205: if __name__ == "__main__":
206:     main()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `data_processing` / 符号 `data_processing`
- Symbol `tokenize_function` / 符号 `tokenize_function`
- Symbol `training_iter_fn` / 符号 `training_iter_fn`
- Symbol `model_training_evaluation` / 符号 `model_training_evaluation`

## Dependencies / 依赖关系
- Python imports: `argparse`, `inspect`, `os`, `sys`, `time`, `datetime`, `datasets`, `transformers`, `torch`, `torch._dynamo`
- Python 导入: `argparse`, `inspect`, `os`, `sys`, `time`, `datetime`, `datasets`, `transformers`, `torch`, `torch._dynamo`
