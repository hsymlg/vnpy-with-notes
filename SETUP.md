# 环境配置指南

## 前置要求

- Python >= 3.10
- macOS 需要先安装 ta-lib 底层库：

```bash
brew install ta-lib
```

## 创建虚拟环境并安装依赖

```bash
cd /Users/hsymlg/PycharmProjects/vnpy-with-notes

# 创建虚拟环境
python3 -m venv .venv

# 激活虚拟环境
source .venv/bin/activate

# 安装全部依赖（含 alpha ML 模块）
pip install -e ".[alpha]"
```

> 安装过程较长，包含 torch、lightgbm 等大型依赖，耐心等待。

## 配置 PyCharm 解释器

1. `Settings` → `Project` → `Python Interpreter`
2. `Add Interpreter` → `Add Local Interpreter`
3. 选 `Existing` → 路径填：

```
/Users/hsymlg/PycharmProjects/vnpy-with-notes/.venv/bin/python
```

4. 确认后 PyCharm 自动重建索引，所有 import 飘红消失。

## 每次开发前激活环境

```bash
source .venv/bin/activate
```

退出环境：

```bash
deactivate
```
