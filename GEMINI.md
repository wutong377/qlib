# Qlib 项目指南

Qlib 是一个由微软开发的面向 AI 的开源量化投资平台。它旨在通过 AI 技术发掘量化投资的潜力，涵盖了从数据处理、模型训练、回测到策略执行的全流程。

## 项目概览

- **核心价值**: 提供全栈式量化投资解决方案，支持多种机器学习范式（监督学习、市场动态建模、强化学习）。
- **主要技术栈**: Python, PyTorch, LightGBM, XGBoost, Cython (用于高性能数据计算), MLflow (实验追踪)。
- **项目结构**:
    - `qlib/`: 核心源码，包含数据提供者、模型接口、回测框架等。
    - `examples/`: 包含各种模型的 Benchmark 和教学 Notebooks。
    - `scripts/`: 数据采集、格式转换和系统工具。
    - `tests/`: 完备的单元测试和集成测试。

## 环境与安装

### 推荐环境
- Python 3.8+ (建议使用 Conda 环境)。
- 在 macOS 上安装时，若遇到 LightGBM 编译问题，需先运行 `brew install libomp`。

### 安装命令
- **从 PyPI 安装**: `pip install pyqlib`
- **从源码安装 (推荐开发使用)**:
    ```bash
    git clone https://github.com/microsoft/qlib.git && cd qlib
    make install  # 会自动处理 Cython 模块的编译
    ```
- **开发环境配置**: `make dev` (安装所有开发及测试依赖)。

## 快速上手

### 1. 数据准备
Qlib 使用高效的二进制格式存储数据。使用以下命令获取示例数据：
```bash
python scripts/get_data.py qlib_data --target_dir ~/.qlib/qlib_data/cn_data --region cn
```

### 2. 运行工作流 (qrun)
Qlib 提供 `qrun` 工具，可通过 YAML 配置文件自动执行整个研发流程（数据加载、模型训练、回测、评估）：
```bash
qrun examples/benchmarks/LightGBM/workflow_config_lightgbm_Alpha158.yaml
```

### 3. 代码内初始化
在 Python 脚本中使用 Qlib 前必须初始化：
```python
import qlib
qlib.init(provider_uri='~/.qlib/qlib_data/cn_data')
```

## 开发规范

- **代码风格**: 严格遵循 `black` (120字符长度), `pylint`, `flake8`。
- **静态检查**: 使用 `mypy` 进行类型检查。
- **性能优化**: 关键的滚动计算逻辑位于 `qlib/data/_libs/`，使用 Cython 编写。
- **实验追踪**: 默认集成 MLflow，运行结果存储在 `mlruns` 目录下。
- **常用开发命令**:
    - `make lint`: 运行所有代码规范检查。
    - `pytest`: 执行单元测试。
    - `make clean`: 清理构建产物和临时文件。

## 关键模块指南

- **Data (qlib.data)**: 提供高性能的数据访问接口（如 `D.features`, `D.calendar`）。
- **Model (qlib.model)**: 模型基类，用户可在此基础上实现自定义 AI 模型。
- **Backtest (qlib.backtest)**: 灵活的交易模拟器，支持多种执行策略。
- **Workflow (qlib.workflow)**: 实验管理模块，负责记录参数、指标和产出。

## 贡献指南
- 提交 PR 前请务必运行 `make lint` 确保代码符合规范。
- 新增功能或修复 Bug 需同步添加对应的测试用例。
- 复杂逻辑请参考 `docs/developer/code_standard_and_dev_guide.rst`。


```mermaid
graph TB
     subgraph 应用层
         DataHelper[DataHelper</br>便捷方法封装]
     end

     subgraph 数据访问层
         DataAccessor[DataAccessor</br>统一访问入口]
     end

     subgraph 读写层
         DataReader[DataReader</br>数据读取]
         DataWriter[DataWriter</br>数据写入]
     end

     subgraph 缓存与索引
         QueryCache[QueryCache</br>查询缓存]
         MetadataIndex[MetadataIndex</br>元数据索引]
         SchemaRegistry[SchemaRegistry</br>Schema注册]
     end

     subgraph 存储层
         Parquet[(Parquet文件</br>按年分区)]
     end

     DataHelper --> DataAccessor
     DataAccessor --> DataReader
     DataAccessor --> DataWriter

     DataReader --> QueryCache
     DataReader --> MetadataIndex
     DataReader --> Parquet

     DataWriter --> MetadataIndex
     DataWriter --> SchemaRegistry
     DataWriter --> Parquet
     