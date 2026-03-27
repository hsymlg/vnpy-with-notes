# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 提供在此代码库中工作的指导。

## 项目简介

VeighNa (vnpy) v4.3.0 是一个面向专业交易员的生产级事件驱动量化交易框架，支持 30+ 交易接口（Gateway）、可选 Qt 图形界面，以及 AI/ML 研究模块（`vnpy.alpha`）。

## 常用命令

### 安装
```bash
# macOS
bash install_osx.sh

# Ubuntu/Linux
bash install.sh

# 所有平台（可编辑安装）
pip install -e .

# 含 alpha（机器学习）依赖
pip install -e .[alpha]
```

### 代码检查与类型检查
```bash
# 代码检查
ruff check .

# 类型检查
mypy vnpy
```

### 测试
```bash
# 运行所有测试
pytest tests/

# 运行单个测试文件
pytest tests/test_alpha101.py
```

### 构建
```bash
uv build
```

## 架构

### 核心设计：事件驱动 + 插件系统

所有组件通过 `EventEngine`（发布/订阅队列）通信。Gateway、App、Engine 均以插件形式注册到 `MainEngine`。

```
MainEngine
├── EventEngine       ← 中央消息总线（队列 + 后台线程）
├── Gateways          ← 券商/交易所接口（BaseGateway 子类）
├── Apps              ← 功能模块：CTA、Portfolio 等（BaseApp 子类）
└── Engines           ← 内部服务：RPC、日志等（BaseEngine 子类）
```

**核心数据流：** Gateway 推送行情数据 → EventEngine → 已注册的策略回调 → 策略发出委托 → MainEngine → Gateway → 券商。

### 模块说明

**`vnpy.event`** — 事件总线核心。`EventEngine` 运行后台队列处理线程和定时器线程。通过 `event_engine.register(EVENT_TYPE, handler)` 注册事件处理函数。

**`vnpy.trader`** — 交易平台核心：
- `engine.py`：`MainEngine` 统筹协调；`BaseEngine`/`BaseApp`/`BaseGateway` 是扩展点
- `object.py`：共享数据结构（`BarData`、`TickData`、`OrderData`、`TradeData`、`PositionData`、`ContractData`），均为 `@dataclass`
- `constant.py`：枚举常量（`Exchange`、`Interval`、`Direction`、`Offset`、`Status`、`OrderType`）
- `gateway.py`：`BaseGateway` — 继承此类以接入新交易接口
- `database.py`：数据库抽象层
- `setting.py`：全局 `SETTINGS` 字典，持久化至 `~/.vntrader/`
- `ui/`：PySide6 Qt 图形界面（可选，框架支持无界面运行）

**`vnpy.alpha`** — AI/ML 量化研究（v4.0 核心特性）：
- `lab.py`：`AlphaLab` — 统一研究工作流（数据 → 特征 → 模型 → 信号）
- `dataset/`：因子/特征工程。`AlphaDataset` 基类；`alpha_101.py`（101 个技术因子）、`alpha_158.py`（158 个 Qlib 风格特征）
- `model/`：ML 模型 — `LassoModel`、`LgbModel`（LightGBM）、`MlpModel`（PyTorch）
- `strategy/backtesting.py`：策略回测引擎

**`vnpy.rpc`** — 基于 ZMQ 的 RPC，用于多进程/分布式部署。

**`vnpy.chart`** — 实时 K 线图表控件（pyqtgraph + PySide6）。

### 并发模型

基于多线程（无 async/await）。`EventEngine` 使用 `queue.Queue` 由后台线程处理。Gateway 通常自行启动线程处理网络 I/O。

### 配置管理

全局配置通过 `vnpy/trader/setting.py` 中的 `SETTINGS` 字典管理，以 JSON 格式持久化至 `~/.vntrader/`（或工作目录下的 `.vntrader/`）。

### 扩展点

- **新交易接口**：继承 `vnpy/trader/gateway.py` 中的 `BaseGateway`
- **新应用/策略模块**：继承 `vnpy/trader/app.py` 中的 `BaseApp`
- **新 ML 模型**：继承 `vnpy/alpha/model/template.py` 中的 `AlphaModel`
- **新因子集**：继承 `vnpy/alpha/dataset/template.py` 中的 `AlphaDataset`

## 类型系统

严格 mypy 配置——所有函数必须有类型注解，不允许隐式 `Any`。`polars` 和 `lightgbm` 因缺少类型桩文件有专项豁免。Ruff 检查规则：B、E、F、UP、W；忽略 E501（行长度限制）。
