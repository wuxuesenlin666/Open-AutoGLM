# Open-AutoGLM

[Read this in English.](./README_en.md)

<div align="center">
<img src=resources/logo.svg width="20%"/>
</div>
<p align="center">
    👋 加入我们的 <a href="resources/WECHAT.md" target="_blank">微信</a> 社区
</p>

## 项目介绍

Phone Agent 是一个基于 AutoGLM 构建的手机端智能助理框架，它能够以多模态方式理解手机屏幕内容，并通过自动化操作帮助用户完成任务。系统通过
ADB（Android Debug Bridge）来控制设备，以视觉语言模型进行屏幕感知，再结合智能规划能力生成并执行操作流程。用户只需用自然语言描述需求，如“打开小红书搜索美食”，Phone
Agent 即可自动解析意图、理解当前界面、规划下一步动作并完成整个流程。系统还内置敏感操作确认机制，并支持在登录或验证码场景下进行人工接管。同时，它提供远程
ADB 调试能力，可通过 WiFi 或网络连接设备，实现灵活的远程控制与开发。

> ⚠️ 本项目仅供研究和学习使用。严禁用于非法获取信息、干扰系统或任何违法活动。请仔细审阅 [使用条款](resources/privacy_policy.txt)。

## 模型下载地址

| Model            | Download Links                                                                                                                               |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| AutoGLM-Phone-9B | [🤗 Hugging Face](https://huggingface.co/zai-org/AutoGLM-Phone-9B)<br>[🤖 ModelScope](https://modelscope.cn/models/ZhipuAI/AutoGLM-Phone-9B) |

## 环境准备

### 1. Python 环境

建议使用 Python 3.10 及以上版本。

### 2. ADB (Android Debug Bridge)

1. 下载官方 ADB [安装包](https://developer.android.com/tools/releases/platform-tools?hl=zh-cn)，并解压到自定义路径
2. 配置环境变量

- MacOS 配置方法：在 `Terminal` 或者任何命令行工具里

  ```bash
  # 假设解压后的目录为 ~/Downlaods/platform-tools。如果不是请自行调整命令。
  export PATH=${PATH}:~/Downloads/platform-tools
  ```

- Windows 配置方法：可参考 [第三方教程](https://blog.csdn.net/x2584179909/article/details/108319973) 进行配置。

### 3. Android 7.0+ 的设备或模拟器，并启用 `开发者模式` 和 `USB 调试`

1. 开发者模式启用：通常启用方法是，找到 `设置-关于手机-版本号` 然后连续快速点击 10
   次左右，直到弹出弹窗显示“开发者模式已启用”。不同手机会有些许差别，如果找不到，可以上网搜索一下教程。
2. USB 调试启用：启用开发者模式之后，会出现 `设置-开发者选项-USB 调试`，勾选启用
3. 部分机型在设置开发者选项以后, 可能需要重启设备才能生效. 可以测试一下: 将手机用USB数据线连接到电脑后, `adb devices`
   查看是否有设备信息, 如果没有说明连接失败.

### 4. 安装 ADB Keyboard（用于文本输入）

下载 [安装包](https://github.com/senzhk/ADBKeyBoard/blob/master/ADBKeyboard.apk) 并在对应的安卓设备中进行安装。
注意，安装完成后还需要到 `设置-输入法` 或者 `设置-键盘列表` 中启用 `ADB Keyboard` 才能生效

## 部署准备工作

### 1. 安装依赖

```bash
pip install -r requirements.txt 
pip install -e .
```

### 2. 配置 ADB

确认 **USB数据线具有数据传输功能**, 而不是仅有充电功能

确保已安装 ADB 并使用 **USB数据线** 连接设备：

```bash
# 检查已连接的设备
adb devices

# 输出结果应显示你的设备，如：
# List of devices attached
# emulator-5554   device
```

### 3. 启动模型服务

1. 下载模型，并按照 `requirements.txt` 中 `For Model Deployment` 章节自行安装推理引擎框架。
2. 通过 SGlang / vLLM 启动，得到 OpenAI 格式服务。这里提供一个 vLLM部署方案，请严格遵循我们提供的启动参数:

- vLLM:

```shell
python3 -m vllm.entrypoints.openai.api_server \
 --served-model-name autoglm-phone-9b \
 --allowed-local-media-path /   \
 --mm-encoder-tp-mode data \
 --mm_processor_cache_type shm \
 --mm_processor_kwargs "{\"max_pixels\":5000000}" \
 --max-model-len 25480  \
 --chat-template-content-format string \
 --limit-mm-per-prompt "{\"image\":10}" \
 --model zai-org/AutoGLM-Phone-9B \
 --port 8000
```

- 该模型结构与 `GLM-4.1V-9B-Thinking` 相同, 关于模型部署的详细内容，你也可以查看 [GLM-V](https://github.com/zai-org/GLM-V)
  获取模型部署和使用指南。

- 运行成功后，将可以通过 `http://localhost:8000/v1` 访问模型服务。 如果您在远程服务器部署模型, 使用该服务器的IP访问模型.

## 免费 GPU 服务器部署方案

如果您没有显存较大的 GPU 服务器，可以考虑使用以下免费或低成本的 GPU 资源来部署 AutoGLM-Phone-9B 模型：

### GPU 资源需求

AutoGLM-Phone-9B 是一个 9B 参数的多模态模型，推荐配置：
- **最低要求**: 24GB 显存 (如 NVIDIA RTX 3090, RTX 4090, A5000)
- **推荐配置**: 40GB+ 显存 (如 NVIDIA A100, A6000)
- **量化后**: 16GB 显存可运行 (使用 INT4/INT8 量化)

### 免费/低成本 GPU 平台推荐

#### 1. Google Colab (推荐新手)

Google Colab 提供免费的 GPU 资源，适合学习和测试：

**免费版特点:**
- GPU: Tesla T4 (16GB 显存) 或 V100 (16GB)
- 每次连续使用最长 12 小时
- 需要定期重新连接
- 适合测试和学习

**Colab Pro/Pro+ (付费):**
- GPU: V100 (16GB) 或 A100 (40GB)
- 更长的运行时间和更好的稳定性
- 月付 $9.99 (Pro) 或 $49.99 (Pro+)

**使用步骤:**
1. 访问 [Google Colab](https://colab.research.google.com/)
2. 创建新笔记本，启用 GPU: `Runtime -> Change runtime type -> GPU`
3. 安装依赖和部署模型
4. 使用 Colab 的公网隧道（如 ngrok）暴露模型服务

**示例代码:**
```python
# 安装依赖
!pip install vllm transformers

# 启动模型服务（建议使用量化以适应 T4 16GB 显存）
!python -m vllm.entrypoints.openai.api_server \
  --model zai-org/AutoGLM-Phone-9B \
  --served-model-name autoglm-phone-9b \
  --allowed-local-media-path / \
  --mm-encoder-tp-mode data \
  --mm_processor_cache_type shm \
  --mm_processor_kwargs "{\"max_pixels\":5000000}" \
  --max-model-len 8192 \
  --chat-template-content-format string \
  --limit-mm-per-prompt "{\"image\":10}" \
  --port 8000 \
  --quantization awq  # 使用量化减少显存占用，降低 max-model-len 减少显存占用
```

#### 2. Kaggle Notebooks

Kaggle 提供稳定的免费 GPU 资源：

**免费版特点:**
- GPU: Tesla P100 (16GB) 或 T4 (16GB)
- 每周 30 小时免费 GPU 时间
- 相对稳定，适合中长时间任务

**使用步骤:**
1. 注册 [Kaggle](https://www.kaggle.com/) 账号
2. 创建 Notebook: `Notebooks -> New Notebook`
3. 启用 GPU: `Settings -> Accelerator -> GPU`
4. 部署模型并暴露服务

#### 3. Hugging Face Spaces (适合 Demo 部署)

Hugging Face 提供免费的模型托管服务：

**特点:**
- 免费版: CPU/小型 GPU
- 付费版: A10G (24GB) 或 A100 (40GB)
- 适合部署 Demo 和长期服务
- 与 Hugging Face 模型库无缝集成

**使用步骤:**
1. 在 [Hugging Face Spaces](https://huggingface.co/spaces) 创建新空间
2. 选择 Gradio 或 Docker 模板
3. 配置硬件: Settings -> Hardware -> GPU
4. 部署模型服务

#### 4. AutoDL / 智星云 / 恒源云 (国内选项)

国内提供低成本 GPU 租赁服务：

**AutoDL (推荐国内用户):**
- 按小时计费，价格低廉（RTX 3090 约 ¥2-3/小时）
- 提供 RTX 3090, A5000, A100 等多种显卡
- 国内访问速度快，支持微信/支付宝支付
- 网址: [https://www.autodl.com/](https://www.autodl.com/)

**智星云:**
- 价格实惠，新用户有优惠
- 提供多种 GPU 选项
- 网址: [https://www.ai-galaxy.cn/](https://www.ai-galaxy.cn/)

**恒源云:**
- 按需付费，灵活计费
- GPU 选项丰富
- 网址: [https://gpushare.com/](https://gpushare.com/)

#### 5. Lightning AI (原 Grid.ai)

提供免费和付费的 GPU 云服务：

**特点:**
- 免费额度: 每月有限的免费 GPU 小时
- 易于部署和管理
- 支持 PyTorch 生态
- 网址: [https://lightning.ai/](https://lightning.ai/)

### 模型量化以减少显存需求

如果 GPU 显存不足，可以使用模型量化技术：

**使用 vLLM 量化 (推荐):**
```bash
# INT8 量化 (约减少 50% 显存)
python -m vllm.entrypoints.openai.api_server \
  --model zai-org/AutoGLM-Phone-9B \
  --served-model-name autoglm-phone-9b \
  --allowed-local-media-path / \
  --mm-encoder-tp-mode data \
  --mm_processor_cache_type shm \
  --mm_processor_kwargs "{\"max_pixels\":5000000}" \
  --max-model-len 25480 \
  --chat-template-content-format string \
  --limit-mm-per-prompt "{\"image\":10}" \
  --port 8000 \
  --quantization int8

# AWQ 4-bit 量化 (约减少 75% 显存)
# 需要先准备 AWQ 量化权重
python -m vllm.entrypoints.openai.api_server \
  --model zai-org/AutoGLM-Phone-9B \
  --served-model-name autoglm-phone-9b \
  --allowed-local-media-path / \
  --mm-encoder-tp-mode data \
  --mm_processor_cache_type shm \
  --mm_processor_kwargs "{\"max_pixels\":5000000}" \
  --max-model-len 25480 \
  --chat-template-content-format string \
  --limit-mm-per-prompt "{\"image\":10}" \
  --port 8000 \
  --quantization awq
```

**预期显存占用:**
- FP16 全精度: ~20GB
- INT8 量化: ~10GB
- INT4/AWQ 量化: ~6GB

### 远程部署最佳实践

1. **选择合适的平台**: 新手推荐 Colab/Kaggle，长期使用推荐国内按时计费平台
2. **使用量化**: 如果显存不足，优先考虑 INT8 或 AWQ 量化
3. **配置网络访问**: 使用 ngrok、frp 或平台提供的公网 IP 暴露服务
4. **定期保存检查点**: 免费平台可能会断连，注意保存工作进度
5. **监控资源使用**: 注意 GPU 使用时间限制和配额

### 故障排查

**显存不足 (OOM):**
- 尝试使用量化: `--quantization int8` 或 `--quantization awq`
- 减小批处理大小: `--max-num-seqs 1`
- 使用更小的上下文窗口: `--max-model-len 8192`

**连接超时:**
- 使用稳定的网络隧道工具 (ngrok, cloudflared)
- 配置合适的超时参数
- 考虑使用国内平台减少延迟

## 使用 AutoGLM

### 命令行

根据你部署的模型, 设置 `--base-url` 和 `--model` 参数. 例如:

```bash
# 交互模式
python main.py --base-url http://localhost:8000/v1 --model "autoglm-phone-9b"

# 指定模型端点
python main.py --base-url http://localhost:8000/v1 "打开美团搜索附近的火锅店"

# 列出支持的应用
python main.py --list-apps
```

### Python API

```python
from phone_agent import PhoneAgent
from phone_agent.model import ModelConfig

# Configure model
model_config = ModelConfig(
    base_url="http://localhost:8000/v1",
    model_name="autoglm-phone-9b",
)

# 创建 Agent
agent = PhoneAgent(model_config=model_config)

# 执行任务
result = agent.run("打开淘宝搜索无线耳机")
print(result)
```

## 远程调试

Phone Agent 支持通过 WiFi/网络进行远程 ADB 调试，无需 USB 连接即可控制设备。

### 配置远程调试

#### 在手机端开启无线调试

确保手机和电脑在同一个WiFi中，如图所示

![开启无线调试](resources/setting.png)

#### 在电脑端使用标准 ADB 命令

```bash

# 通过 WiFi 连接, 改成手机显示的 IP 地址和端口
adb connect 192.168.1.100:5555

# 验证连接
adb devices
# 应显示：192.168.1.100:5555    device
```

### 设备管理命令

```bash
# 列出所有已连接设备
adb devices

# 连接远程设备
adb connect 192.168.1.100:5555

# 断开指定设备
adb disconnect 192.168.1.100:5555

# 指定设备执行任务
python main.py --device-id 192.168.1.100:5555 --base-url http://localhost:8000/v1 --model "autoglm-phone-9b" "打开抖音刷视频"
```

### Python API 远程连接

```python
from phone_agent.adb import ADBConnection, list_devices

# 创建连接管理器
conn = ADBConnection()

# 连接远程设备
success, message = conn.connect("192.168.1.100:5555")
print(f"连接状态: {message}")

# 列出已连接设备
devices = list_devices()
for device in devices:
    print(f"{device.device_id} - {device.connection_type.value}")

# 在 USB 设备上启用 TCP/IP
success, message = conn.enable_tcpip(5555)
ip = conn.get_device_ip()
print(f"设备 IP: {ip}")

# 断开连接
conn.disconnect("192.168.1.100:5555")
```

### 远程连接问题排查

**连接被拒绝：**

- 确保设备和电脑在同一网络
- 检查防火墙是否阻止 5555 端口
- 确认已启用 TCP/IP 模式：`adb tcpip 5555`

**连接断开：**

- WiFi 可能断开了，使用 `--connect` 重新连接
- 部分设备重启后会禁用 TCP/IP，需要通过 USB 重新启用

**多设备：**

- 使用 `--device-id` 指定要使用的设备
- 或使用 `--list-devices` 查看所有已连接设备

## 配置

### 自定义SYSTEM PROMPT

直接修改配置文件 `phone_agent/config/prompts.py`

1. 可以通过注入system prompt来增强模型在特定领域的能力
2. 可以通过注入app名称禁用某些app

### 环境变量

| 变量                      | 描述        | 默认值                        |
|-------------------------|-----------|----------------------------|
| `PHONE_AGENT_BASE_URL`  | 模型 API 地址 | `http://localhost:8000/v1` |
| `PHONE_AGENT_MODEL`     | 模型名称      | `autoglm-phone-9b`         |
| `PHONE_AGENT_MAX_STEPS` | 每个任务最大步数  | `100`                      |
| `PHONE_AGENT_DEVICE_ID` | ADB 设备 ID | (自动检测)                     |

### 模型配置

```python
from phone_agent.model import ModelConfig

config = ModelConfig(
    base_url="http://localhost:8000/v1",
    api_key="EMPTY",  # API 密钥（如需要）
    model_name="autoglm-phone-9b",  # 模型名称
    max_tokens=3000,  # 最大输出 token 数
    temperature=0.1,  # 采样温度
    frequency_penalty=0.2,  # 频率惩罚
)
```

### Agent 配置

```python
from phone_agent.agent import AgentConfig

config = AgentConfig(
    max_steps=100,  # 每个任务最大步数
    device_id=None,  # ADB 设备 ID（None 为自动检测）
    verbose=True,  # 打印调试信息（包括思考过程和执行动作）
)
```

### Verbose 模式输出

当 `verbose=True` 时，Agent 会在每一步输出详细信息：

```
==================================================
💭 思考过程:
--------------------------------------------------
当前在系统桌面，需要先启动小红书应用
--------------------------------------------------
🎯 执行动作:
{
  "_metadata": "do",
  "action": "Launch",
  "app": "小红书"
}
==================================================

... (执行动作后继续下一步)

==================================================
💭 思考过程:
--------------------------------------------------
小红书已打开，现在需要点击搜索框
--------------------------------------------------
🎯 执行动作:
{
  "_metadata": "do",
  "action": "Tap",
  "element": [500, 100]
}
==================================================

🎉 ================================================
✅ 任务完成: 已成功搜索美食攻略
==================================================
```

这样可以清楚地看到 AI 的推理过程和每一步的具体操作。

## 支持的应用

Phone Agent 支持 50+ 款主流中文应用：

| 分类   | 应用              |
|------|-----------------|
| 社交通讯 | 微信、QQ、微博        |
| 电商购物 | 淘宝、京东、拼多多       |
| 美食外卖 | 美团、饿了么、肯德基      |
| 出行旅游 | 携程、12306、滴滴出行   |
| 视频娱乐 | bilibili、抖音、爱奇艺 |
| 音乐音频 | 网易云音乐、QQ音乐、喜马拉雅 |
| 生活服务 | 大众点评、高德地图、百度地图  |
| 内容社区 | 小红书、知乎、豆瓣       |

运行 `python main.py --list-apps` 查看完整列表。

## 可用操作

Agent 可以执行以下操作：

| 操作           | 描述              |
|--------------|-----------------|
| `Launch`     | 启动应用            |  
| `Tap`        | 点击指定坐标          |
| `Type`       | 输入文本            |
| `Swipe`      | 滑动屏幕            |
| `Back`       | 返回上一页           |
| `Home`       | 返回桌面            |
| `Long Press` | 长按              |
| `Double Tap` | 双击              |
| `Wait`       | 等待页面加载          |
| `Take_over`  | 请求人工接管（登录/验证码等） |

## 自定义回调

处理敏感操作确认和人工接管：

```python
def my_confirmation(message: str) -> bool:
    """敏感操作确认回调"""
    return input(f"确认执行 {message}？(y/n): ").lower() == "y"


def my_takeover(message: str) -> None:
    """人工接管回调"""
    print(f"请手动完成: {message}")
    input("完成后按回车继续...")


agent = PhoneAgent(
    confirmation_callback=my_confirmation,
    takeover_callback=my_takeover,
)
```

## 示例

查看 `examples/` 目录获取更多使用示例：

- `basic_usage.py` - 基础任务执行
- 单步调试模式
- 批量任务执行
- 自定义回调

## 二次开发

### 配置开发环境

二次开发需要使用开发依赖：

```bash
pip install -e ".[dev]"
```

### 运行测试

```bash
pytest tests/
```

### 完整项目结构

```
phone_agent/
├── __init__.py          # 包导出
├── agent.py             # PhoneAgent 主类
├── adb/                 # ADB 工具
│   ├── connection.py    # 远程/本地连接管理
│   ├── screenshot.py    # 屏幕截图
│   ├── input.py         # 文本输入 (ADB Keyboard)
│   └── device.py        # 设备控制 (点击、滑动等)
├── actions/             # 操作处理
│   └── handler.py       # 操作执行器
├── config/              # 配置
│   ├── apps.py          # 支持的应用映射
│   └── prompts.py       # 系统提示词
└── model/               # AI 模型客户端
    └── client.py        # OpenAI 兼容客户端
```

## 常见问题

我们列举了一些常见的问题，以及对应的解决方案：

### 设备未找到

尝试通过重启 ADB 服务来解决：

```bash
adb kill-server
adb start-server
adb devices
```

### 文本输入不工作

1. 确保设备已安装 ADB Keyboard
2. 在设置 > 系统 > 语言和输入法 > 虚拟键盘 中启用
3. Agent 会在需要输入时自动切换到 ADB Keyboard

### 截图失败（黑屏）

这通常意味着应用正在显示敏感页面（支付、密码、银行类应用）。Agent 会自动检测并请求人工接管。

### 引用

如果你觉得我们的工作有帮助，请引用以下论文：

```bibtex
@article{liu2024autoglm,
  title={Autoglm: Autonomous foundation agents for guis},
  author={Liu, Xiao and Qin, Bo and Liang, Dongzhu and Dong, Guang and Lai, Hanyu and Zhang, Hanchen and Zhao, Hanlin and Iong, Iat Long and Sun, Jiadai and Wang, Jiaqi and others},
  journal={arXiv preprint arXiv:2411.00820},
  year={2024}
}
@article{xu2025mobilerl,
  title={MobileRL: Online Agentic Reinforcement Learning for Mobile GUI Agents},
  author={Xu, Yifan and Liu, Xiao and Liu, Xinghan and Fu, Jiaqi and Zhang, Hanchen and Jing, Bohao and Zhang, Shudan and Wang, Yuting and Zhao, Wenyi and Dong, Yuxiao},
  journal={arXiv preprint arXiv:2509.18119},
  year={2025}
}
```
