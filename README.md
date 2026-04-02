# 🤖 离线模型下载与配置指南 (README_MODELS)

为了让翻译器实现 **「0 秒极速开启」** 和 **「100% 离线使用」**，建议您手动下载 `large-v3` 模型并按以下方式存放。

---

## 0. 运行环境前置要求（重要）
如果希望启用 NVIDIA GPU 加速，请确保环境与项目一致：

- Python `3.10`
- `torch==2.5.1+cu121` / `torchaudio==2.5.1+cu121` / `torchvision==0.20.1+cu121`
- CUDA `12.1`
- cuDNN `8.9`（需确保系统可找到 `cudnn_ops_infer64_8.dll`）

说明：
- PyTorch Python 包版本已在 `requirements.txt` 中锁定。
- CUDA / cuDNN 属于系统级运行库，不能仅靠 `requirements.txt` 自动安装，需手动安装并配置 PATH。

### 一键检查与安装命令（PowerShell）
请在项目根目录执行以下命令：
这是 NVIDIA 官方 cuDNN 下载页（需要登录 NVIDIA 账号）：

https://developer.nvidia.com/cudnn-downloads
```bash
# 1) 确认 Python 版本与解释器路径
python -V
python -c "import sys; print(sys.executable)"

# 2) 安装/重装 GPU 版 PyTorch (CUDA 12.1)
python -m pip uninstall -y torch torchvision torchaudio
python -m pip install --upgrade pip
python -m pip install torch==2.5.1+cu121 torchvision==0.20.1+cu121 torchaudio==2.5.1+cu121 --extra-index-url https://download.pytorch.org/whl/cu121

# 3) 安装项目依赖
python -m pip install -r requirements.txt

# 4) 检查系统是否能找到关键 CUDA/cuDNN DLL
where cudnn_ops_infer64_8.dll
where cublas64_12.dll

# 5) 校验 PyTorch CUDA 状态
python -c "import torch; print('torch=', torch.__version__); print('cuda_available=', torch.cuda.is_available()); print('torch_cuda=', torch.version.cuda)"

# 6) 启动项目
python app.py
```

若第 4 步找不到 `cudnn_ops_infer64_8.dll`，请将 cuDNN 8.9 的 DLL 放入：
`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\bin`

---

## 1. 下载模型文件
请访问下方镜像站，下载 **所有** 文件（约 3GB）：

👉 **[模型下载地址 (HF-Mirror 加速)](https://hf-mirror.com/Systran/faster-whisper-large-v3)**

### 必下文件清单：
*   ✅ `model.bin` (模型核心权重)
*   ✅ `config.json` (配置文件)
*   ✅ `vocabulary.txt` / `tokenizer.json` (分词器文件)

---

## 2. 存放目录结构
下载完成后，请在项目根目录下构建如下结构（**极其重要**）：

```text
视频翻译/
├── app.py
├── models/
│   └── large-v3/
│       ├── model.bin
│       ├── config.json
│       ├── vocabulary.txt
│       └── ... (其他下载的文件)
└── ...
```

---

## 3. 验证成功
放好后，直接启动 `start.bat` 并点击翻译。
如果看到日志显示：
> **`[INFO] 检测到本地离线仓库: .../models/large-v3`**
> **`[INFO] 正在 0 秒加载本地模型...`**

恭喜您！您的翻译器已进入 **「极速模式」**，且不再消耗任何网络流量。

---
*注：本程序支持全盘符（D/E 盘）自适应运行，只需确保模型夹在程序旁边的 models 文件夹内即可。*

---

## 4. 说话人分离 (Speaker Diarization) 模型授权 🔐

本程序内置了 **WhisperX** 的辅助对齐与角色识别功能，这需要使用 `pyannote` 相关的模型。
由于 `pyannote` 的政策，您需要手动在 Hugging Face 网页上接受其用户协议。

### 步骤 A：访问并接受协议 (必须完成)
即使您不主动下载这两个模型，也**必须**登录 Hugging Face 账号并在这两个页面上点击 **"Accept"** 或 **"Agree"**：

1.  👉 **[Segmentation 3.0 授权页](https://huggingface.co/pyannote/segmentation-3.0)**
2.  👉 **[Speaker Diarization 3.1 授权页](https://huggingface.co/pyannote/speaker-diarization-3.1)**

### 步骤 B：配置 Access Token
1. 在您的 [Hugging Face Settings -> Access Tokens](https://huggingface.co/settings/tokens) 中创建一个 `READ` 权限的 Token。
2. 将此 Token 填写到项目根目录的 `.env` 文件中：
   ```dotenv
   HF_TOKEN=hf_xxxxxxxxxxxxxxxxx
   ```

---
**祝您的翻译工作一切顺利！如有遇到加载超时，请确认全局/终端代理已开启且能访问 huggingface.co。**

---

## 5. 角色声音克隆 (GPT-SoVITS) 配置 🎙️

本程序支持使用 **GPT-SoVITS** 实现基于原视频音色的 1:1 跨语言克隆。

### 启用步骤：
1.  **下载整合包**：请根据您的网络环境选择以下任一地址下载 Windows 一键运行包：
    *   [官方直链 (Hugging Face)](https://huggingface.co/lj1995/GPT-SoVITS-windows-package/resolve/main/GPT-SoVITS-v2pro-20250604.7z?download=true)
    *   [国内加速直链 (ModelScope 魔搭社区)](https://www.modelscope.cn/models/FlowerCry/gpt-sovits-7z-pacakges/resolve/master/GPT-SoVITS-v2pro-20250604.7z)
2.  **解压服务**：将下载好的 `.7z` 文件解压到**不包含中文或空格**的路径（例如 `D:\GPT-SoVITS`）。
3.  **启动 API 服务**：在解压目录运行 **`go-api.bat`**。
    *   默认地址通常为：`http://127.0.0.1:9880`
3.  **配置程序**：
    *   在「系统设置」中，将 **TTS 引擎** 切换为 `GPT-SoVITS`。
    *   填入正确的「本地服务地址」。

### 常见问题：
*   **连接失败**：请检查 SoVITS 的 CMD 窗口是否已显示 `Running on http://127.0.0.1:9880`。
*   **无参考音频**：程序会自动从原视频中提取 5~10 秒的纯净人声作为参考，无需手动准备。

---

## 6. 核心支持模型清单 (Hub & Align) 🗺️

如果您需要手动部署到全新环境，或者通过代理/镜像站手动下载模型，请按以下列表操作：

### 👥 说话人识别 (Diarization) - 指向项目 `hub` 目录
*由于 Pyannote 政策，手动下载仍需在 Hugging Face 网页端点选 "Accept" 授权。*
1.  👉 **[Speaker Diarization 3.1](https://hf-mirror.com/pyannote/speaker-diarization-3.1)**
2.  👉 **[Segmentation 3.0](https://hf-mirror.com/pyannote/segmentation-3.0)**

### ⚧️ 性别与年龄识别 (Gender ID) - 本地路径 `models/audeering_gender`
这是我们目前使用的**全离线本地化**性别引擎：
1.  👉 **[AudEeERING Wav2Vec2](https://hf-mirror.com/audeering/wav2vec2-large-robust-6-ft-age-gender)**

### 📏 毫秒级对齐 (Alignment) - 本地路径 `models/align`
1.  👉 **[Wav2Vec2 Chinese Alignment](https://hf-mirror.com/jonatasgrosman/wav2vec2-large-xlsr-53-chinese-zh-cn)**

### 🗣️ 配音引擎资产 (TTS Assets / Amphion)
在使用 IndexTTS2 配音引擎时，系统依托于以下声码器与掩码模型：
1.  👉 **[MaskGCT 掩码模型](https://hf-mirror.com/amphion/MaskGCT)**
2.  👉 **[BigVGAN 高保真声码器](https://hf-mirror.com/nvidia/bigvgan_v2_22khz_80band_256x)**

---
*注：下载到本地后的存放目录请参考本指南第 2 节。Hub 模式模型存放至 `models/hub`，其他模型建议存放至各自对应的子目录。*
