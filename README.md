# 3D高斯溅射项目操作指南 (README)

## 1. 环境配置

### 1.1 系统要求
- **操作系统**: Windows 10/11
- **GPU**: NVIDIA显卡 (RTX 20系列及以上)
- **软件**:
    - **Visual Studio 2019**: 需安装 "使用C++的桌面开发" 组件
    - **CUDA Toolkit 11.8**: 版本需与PyTorch匹配
    - **Python 3.7+**: 推荐使用Anaconda或Miniconda

### 1.2 安装依赖
1.  **创建Conda环境**:
    ```bash
    conda create --name gaussian_splatting python=3.7
    conda activate gaussian_splatting
    ```

2.  **安装PyTorch**:
    ```bash
    pip install torch==1.12.1+cu113 torchvision==0.13.1+cu113 -f https://download.pytorch.org/whl/torch_stable.html
    ```

3.  **安装其他依赖**:
    ```bash
    pip install -r requirements.txt
    ```

### 1.3 编译CUDA扩展
项目包含自定义的CUDA扩展，需要手动编译：
```bash
python setup.py install
```
或者运行提供的批处理脚本来编译所有扩展：
```bash
.\compile_extensions.bat
```
编译成功后，`submodules` 目录下会生成对应的 `.pyd` 文件。

## 2. 数据准备

### 2.1 数据格式
项目使用 **COLMAP** 格式的数据。如果您的数据不是此格式，需要进行转换。

### 2.2 数据集结构
将您的图片放入 `data/` 目录，例如 `data/my_dataset/images/`。

### 2.3 使用 `convert.py` 自动处理
运行以下命令，自动调用COLMAP进行特征提取和稀疏重建，生成项目所需的数据集：
```bash
# 切换到3d目录
cd C:\Users\Administrator\Desktop\ML\3d

# 运行转换脚本
python convert.py -s C:\path\to\your\image\folder
```
- `-s`: 原始图片所在目录
转换成功后，会在 `C:\path\to\your\image\folder` 目录下生成 `distorted`、`sparse` 等文件夹。

## 3. 模型训练

### 3.1 训练命令
使用 `train.py` 脚本进行训练：
```bash
# 确保在 3d 目录下
cd C:\Users\Administrator\Desktop\ML\3d

# 开始训练
python train.py -s [SOURCE_PATH] -m [MODEL_PATH] --eval
```

**参数说明**:
- `-s, --source_path [SOURCE_PATH]`: **数据集路径**，指向 `convert.py` 生成的COLMAP格式数据目录 (例如: `C:\Users\Administrator\Desktop\ML\c7c3ef8e-e_new`)
- `-m, --model_path [MODEL_PATH]`: **模型输出路径**，用于保存训练结果 (例如: `./output/my_experiment`)
- `--eval`: **启用评估**，训练过程中会在测试集上进行评估，并保存渲染结果。

**示例**:
```bash
python train.py -s C:\Users\Administrator\Desktop\ML\c7c3ef8e-e_new -m ./output/d0ae14ef-7 --eval
```

### 3.2 监控训练过程
训练过程中，可以使用Tensorboard实时监控：
```bash
# 启动Tensorboard服务
tensorboard --logdir=./output/d0ae14ef-7 --port=6006

# 在浏览器中打开
http://localhost:6006
```
您可以查看 **损失曲线**、**PSNR**、**SSIM** 等指标。

## 4. 模型测试与评估

### 4.1 渲染测试集
训练完成后，使用 `render.py` 脚本渲染测试集图像，用于评估：
```bash
python render.py -m [MODEL_PATH]
```
- `-m`: 已训练好的模型路径 (例如: `./output/d0ae14ef-7`)
渲染结果会保存在模型目录下的 `test/` 子目录中。

### 4.2 计算评价指标
使用 `metrics.py` 脚本计算定量评价指标：
```bash
python metrics.py -m [MODEL_PATH]
```
- `-m`: 已训练好的模型路径 (例如: `./output/d0ae14ef-7`)
脚本会自动计算 **PSNR**、**SSIM** 和 **LPIPS** 指标。

## 5. 结果可视化

### 5.1 使用SIBR Viewer查看3D模型
SIBR Viewer可以交互式地查看训练好的3D高斯点云模型：

1.  **进入 `bin` 目录**:
    ```powershell
    cd C:\Users\Administrator\Desktop\ML\3d\bin
    ```

2.  **启动Viewer**:
    ```powershell
    .\SIBR_gaussianViewer_app.exe -m ..\output\d0ae14ef-7
    ```
    - `-m`: 模型路径 (相对于 `bin` 目录)

**交互操作**:
- **鼠标左键**: 旋转视角
- **鼠标滚轮**: 缩放视角
- **鼠标右键**: 平移视角

### 5.2 查看渲染视频
1.  **渲染图片序列**:
    使用 `render.py` 渲染所有测试集或训练集图片。
2.  **合成视频**:
    使用 `ffmpeg` 等工具将渲染出的图片序列合成为视频：
    ```bash
    ffmpeg -framerate 30 -i path/to/renders/%04d.png -c:v libx264 -pix_fmt yuv420p output.mp4
    ```

---
*本指南提供了从环境配置到训练、测试和可视化的完整流程，希望能帮助您顺利完成实验。* 
