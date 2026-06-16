# Computer Vision Index

> 如果你是第一次进入本目录，先读 [`README.md`](./README.md) 建立方向感；如果你已经知道想找什么，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```
cv/
├── 01-image-fundamentals/              # 图像基础
│   ├── image-processing-basics/
│   │   ├── filtering/
│   │   └── edge-detection/
│   └── feature-extraction/
│
├── 02-image-classification/            # 图像分类
│   ├── classic-backbones/
│   ├── efficient-models/
│   └── training-tricks/
│
├── 03-detection-and-segmentation/      # 目标检测与分割
│   ├── two-stage-detectors/
│   ├── one-stage-detectors/
│   ├── instance-segmentation/
│   ├── semantic-segmentation/
│   └── panoptic-segmentation/
│
├── 04-video-and-3d-vision/             # 视频与3D视觉
│   ├── video-understanding/
│   ├── optical-flow/
│   ├── 3d-reconstruction/
│   └── nerf-and-3d-gaussian-splatting/
│
├── 05-generative-and-multimodal/       # 生成与多模态
│   ├── image-generation/
│   ├── text-to-image/
│   │   └── dall-e/
│   └── multimodal-models/
│
├── 06-foundation-models/               # 自监督与基础模型
│   ├── contrastive-learning/
│   ├── masked-image-modeling/
│   └── vision-foundation-models/
│
└── 07-applications-and-tools/          # 应用与工具
    ├── ocr/
    ├── face-recognition/
    └── deployment-and-optimization/
```

## 按问题找入口

- 想看图像处理基础或特征提取：先看 `01-image-fundamentals/`
- 想看图像分类骨干网络或训练技巧：先看 `02-image-classification/`
- 想看目标检测、分割（实例/语义/全景）或跟踪：先看 `03-detection-and-segmentation/`
- 想看视频理解、光流、3D 重建、NeRF 或 3D Gaussian Splatting：先看 `04-video-and-3d-vision/`
- 想看图像生成、文生图或多模态模型：先看 `05-generative-and-multimodal/`
- 想看对比学习、掩码图像建模或视觉基础模型：先看 `06-foundation-models/`
- 想看 OCR、人脸识别、部署优化等工程化应用：先看 `07-applications-and-tools/`

## 按阅读目标找入口

- 想查经典分类网络结构对比：`02-image-classification/classic-backbones/`
- 想找检测/分割任务的模型选型：`03-detection-and-segmentation/`
- 想了解生成模型在视觉中的实现：`05-generative-and-multimodal/image-generation/`

## 按子目录定位

- `01-image-fundamentals/`：图像处理基础、特征提取
- `02-image-classification/`：经典骨干网络、高效模型、训练技巧
- `03-detection-and-segmentation/`：目标检测与分割（双阶段/单阶段/实例/语义/全景）
- `04-video-and-3d-vision/`：视频理解、光流、3D重建、NeRF 与 3D Gaussian Splatting
- `05-generative-and-multimodal/`：图像生成、文生图、多模态模型
- `06-foundation-models/`：对比学习、掩码图像建模、视觉基础模型
- `07-applications-and-tools/`：OCR、人脸识别、部署优化等工程化应用
