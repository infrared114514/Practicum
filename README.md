# A3:偏好数据构造

## 模块概述
本模块是大模型偏好对齐（DPO）微调训练流水线的数据筛选。在大型语言模型（LLM）的微调过程中，下游的偏好优化算法（如 DPO）高度依赖高质量的正负样本对 `(instruction, chosen, rejected)`。本模块的核心任务是承接原始的工业级 Parquet 数据集，不仅通过严格的过滤机制剥离残缺不全的脏数据，更创新性地引入了基于规则的数据退化增强算法，在不消耗人工标注成本的前提下，批量合成负样本（缺陷代码），为下游团队提供规模更大、健壮性更强的偏好对齐数据集。

## 文件结构与输入配置
```text
dpo/
├── configs/
│   └── environment.yml       # 运行环境配置文件
├── data/
│   ├── .gitkeep              # 占位文件
│   ├── code_dpo_train.json   # 训练集
│   ├── code_effect_test.json # 测试集
│   └── dataset_info.json     # 数据集注册表
├── scripts/
│   └── prepare_data.sh       # 模块启动Shell脚本
└── prepare_dpo_data.py       # 数据清洗与合成
source_data/                  # 存放源数据的文件夹
```

## 整体流程与模块边界
* **输入边界**：从线下存储中读取原始的二进制大文件 Parquet 数据集。
* **逻辑**：
    1.  **数据清洗**：解析字段并进行完整性校验，剔除题目或正确答案缺失的残缺行。
    2.  **数据扩容与退化合成**：以 40% 的随机概率触发合成引擎，对高质量正样本进行基于规则的语法与逻辑退化（如随机剥离冒号、反转边界条件判断符、篡改变量名），从而伪造出高质量的错误偏好数据（`rejected`）。
* **输出边界**：在本地指定目录输出符合微调框架标准的 `code_dpo_train.json`（训练偏好对）、`code_effect_test.json`（测试集）以及用于大模型微调数据注册的 `dataset_info.json`。

## 环境、模型与数据依赖
* **开发环境**：Python 3.10+
* **核心依赖库**：
    ```bash
    pandas >= 1.5.0
    pyarrow >= 9.0.0
    ```
* **数据依赖**：py-dpo-v0.1 目录下的 Parquet 文件。
* **数据集地址**：https://huggingface.co/datasets/jondurbin/py-dpo-v0.1

## 使用方法
* 在py-dpo-v0.1 目录下下载 Parquet 文件后在终端输入如下指令
* bash dpo/scripts/prepare_data.sh
* 之后会在dpo/data 目录下生成code_effect_test.json、code_dpo_train.json、dataset_info.json
