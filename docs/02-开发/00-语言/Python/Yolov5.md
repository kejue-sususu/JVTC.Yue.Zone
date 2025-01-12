# Yolov5

:::info
YOLOv5🚀是一个在COCO数据集上预训练的物体检测架构和模型系列，它代表了Ultralytics对未来视觉AI方法的公开研究，
其中包含了在数千小时的研究和开发中所获得的经验和最佳实践。

[GitHub](https://github.com/ultralytics/yolov5/blob/master/README.zh-CN.md)
:::

## 本地部署

```sh
git clone https://github.com/ultralytics/yolov5.git
cd yolov5
conda create --name yolov5 python=3.11
conda activate yolov5
pip install -i http://mirrors.aliyun.com/pypi/simple/ -r ./requirements.txt --trusted-host mirrors.aliyun.com
# pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r ./requirements.txt
```

## 运行检测

```text
Run YOLOv5 detection inference on images, videos, directories, globs, YouTube, webcam, streams, etc.

Usage - sources:
    $ python detect.py --weights yolov5s.pt --source 0                               # webcam
                                                     img.jpg                         # image
                                                     vid.mp4                         # video
                                                     screen                          # screenshot
                                                     path/                           # directory
                                                     list.txt                        # list of images
                                                     list.streams                    # list of streams
                                                     'path/*.jpg'                    # glob
                                                     'https://youtu.be/Zgi9g1ksQHc'  # YouTube
                                                     'rtsp://example.com/media.mp4'  # RTSP, RTMP, HTTP stream

Usage - formats:
    $ python detect.py --weights yolov5s.pt                 # PyTorch
                                 yolov5s.torchscript        # TorchScript
                                 yolov5s.onnx               # ONNX Runtime or OpenCV DNN with --dnn
                                 yolov5s_openvino_model     # OpenVINO
                                 yolov5s.engine             # TensorRT
                                 yolov5s.mlmodel            # CoreML (macOS-only)
                                 yolov5s_saved_model        # TensorFlow SavedModel
                                 yolov5s.pb                 # TensorFlow GraphDef
                                 yolov5s.tflite             # TensorFlow Lite
                                 yolov5s_edgetpu.tflite     # TensorFlow Edge TPU
                                 yolov5s_paddle_model       # PaddlePaddle

```

## 创建自己的数据集

参考：[Train Custom Data - Ultralytics YOLOv8 Docs](https://docs.ultralytics.com/yolov5/tutorials/train_custom_data)
| [Tips for Best Training Results - Ultralytics YOLOv8 Docs](https://docs.ultralytics.com/yolov5/tutorials/tips_for_best_training_results/)

### 数据标注

#### 使用 Label Studio

[LabelStudio](https://labelstud.io/)

通过 Anaconda 安装：

```shell
conda create --name label-studio python=3.10
conda activate label-studio
pip install label-studio
label-studio start
```

然后浏览器访问 <http://localhost:8080/>

#### 使用 LabelImg

[LabelImg](https://github.com/HumanSignal/labelImg)

## 使用 GPU 训练自己的数据集

### 数据集结构

参考 [`coco.yaml`](https://github.com/ultralytics/yolov5/blob/master/data/coco.yaml)

```txt title="目录结构"
 ├── yolov5
 └── datasets
     └── my-datasets
         ├── images              # 图片
         |   ├── 01.jpg
         |   ├── 02.jpg
         |   ├── 03.jpg
         |   └── ...
         ├── labels              # 标签
         |   ├── classes.txt     # 标签列表
         |   ├── 01.txt
         |   ├── 02.txt
         |   ├── 03.txt
         |   └── ...
         └── my-datasets.yaml    # 训练配置
```

```yaml title="my-datasets.yaml"
# 数据集根目录
path: ../datasets/my_datasets

# 训练数据集目录
train: # txt 文件或相对 path 的目录

# 验证数据集目录
val: # txt 文件或相对 path 的目录

# 测试数据集目录
test: # txt 文件或相对 path 的目录

nc: # 标签总数
names: # 标签数组
```

### NVIDIA CUDA

首先确保自己有一块支持 CUDA 的 GPU。
[查看自己的 GPU 是否支持 CUDA](https://developer.nvidia.com/cuda-gpus)。

下载安装 [CUDA toolkit](https://developer.nvidia.com/cuda-downloads)。
安装完成后可能需要配置 `Path` 环境变量（看看自己的安装位置）：`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\bin`。

使用 `nvcc -V` 查看 CUDA 版本：

```text {1}
$ nvcc.exe -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2023 NVIDIA Corporation
Built on Mon_Apr__3_17:36:15_Pacific_Daylight_Time_2023
Cuda compilation tools, release 12.1, V12.1.105
Build cuda_12.1.r12.1/compiler.32688072_0
```

查看自己安装的 torch：

```ps {1}
(yolov5) PS D:\yolo\v5> conda list | findstr torch
torch                     2.0.1                    pypi_0    pypi
torchvision               0.15.2                   pypi_0    pypi
```

按照 [本地部署](#本地部署) 安装的是使用 CPU 计算的 torch，现在需要卸载它：

```sh
pip uninstall torch
```

参考 [PyTorch 官网](https://pytorch.org/get-started/locally/) 下载安装使用 CUDA 计算的 torch：

```ps
# 文件很大，你忍一下（2.49G）
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia

# 也可以使用 pip 安装（2.6G）
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# 不知道为啥少个包↓
pip install chardet

# 训练过程中提示需要的包（但不装也能跑）↓
# pip install clearml comet_ml
```

#### Error #15: Initializing libiomp5md.dll, but found libiomp5md.dll already initialized.

```text
OMP: Error #15: Initializing libiomp5md.dll, but found libiomp5md.dll already initialized.
OMP: Hint This means that multiple copies of the OpenMP runtime have been linked into the program. That is dangerous, since it can degrade performance or cause incorrect results. The best thing to do is to ensure that only a single OpenMP runtime is linked into the process, e.g. by avoiding static linking of the OpenMP runtime in any library. As an unsafe, unsupported, undocumented workaround you can set the environment variable KMP_DUPLICATE_LIB_OK=TRUE to allow the program to continue to execute, but that may cause crashes or silently produce incorrect results. For more information, please see http://www.intel.com/software/products/support/.
```

设置环境变量：

`KMP_DUPLICATE_LIB_OK` : `TRUE`

### AMD ROCm

[官方网站](https://www.amd.com/zh-hans/graphics/servers-solutions-rocm-ml)
| [官方文档](https://rocm.docs.amd.com/en/latest/)
| [GPU 与系统兼容性（Linux）](https://rocm.docs.amd.com/en/latest/release/gpu_os_support.html)
| [GPU 与系统支持（Windows）](https://rocm.docs.amd.com/en/latest/release/windows_support.html)
| [GitHub](https://github.com/RadeonOpenCompute/ROCm)

综上所述目前 PyTorch 2.x 并不支持在 Windows 上使用 ROCm。

#### DirectML

:::caution
Torch-directml 软件包仅支持 PyTorch 1.13。
:::

参考：[在 Windows 上使用 DirectML 启用 PyTorch](https://learn.microsoft.com/en-us/windows/ai/directml/gpu-pytorch-windows)
| [在 WSL 2 上通过 DirectML 启用 PyTorch](https://learn.microsoft.com/zh-cn/windows/ai/directml/gpu-pytorch-wsl)

### 开始训练

```ps
python train.py --data coco.yaml --epochs 300 --weights '' --cfg yolov5n.yaml  --batch-size 128
                                                                 yolov5s                    64
                                                                 yolov5m                    40
                                                                 yolov5l                    24
                                                                 yolov5x                    16
```
