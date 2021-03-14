---
title: 数字图像篡改取证
date: 2021-01-16 11:55:34
tags:
- forensic
- cv
cover: "https://w.wallhaven.cc/full/dp/wallhaven-dppqeo.png"
---

# 阅读文献

## 术语

| term | explanation |
| ---- | ----------- |
| pixel | 像素，图像的基本单位，一张图片是由一个个pixel组成 |
| patch | 块，每个patch都是由好多个pixel组成的 |

## 🧨Forensic Similarity for Digital Images

### 运行项目

项目地址：https://gitlab.com/MISLgit/forensic-similarity-for-digital-images

### Abstract

- 目的：**forensic similarity**➡**two image patches** contain the same forensic trace or different **forensic traces**
- 应用
  - forgery detection 伪造检测
  - ❓localization 本地化 
  - ❓database consistency verification 数据库一致性检测
- 方法
  - 将成对的图像块映射到一个分数
  - 基于CNN的**特征提取器feature extractor** + 相似性网络(三层神经网络)
- 评估
  - determining whether two image patches were captured by the same or different camera model
  - were manipulated by the same or different editing operation, and
  - were manipulated by the same or different manipulation parameter, given a particular editing operation.
- 原理
  - detect the visually imperceptible **traces or fingerprints** that are intrinsically introduced by **a particular processing operation**

### INTRODUCTION

- 前人缺点
  - [需要先验]require **prior** training examples from a **particular** forensic trace / many deep learning systems assume a **closed** set of forensic traces
    - close set的缺点:系统会将这个新的“未知”trace错误分类为Y(用于训练系统的已知法医痕迹的空间,包含camera models和editing operations)中的“已知”trace,从而检测错误
- [过于精确]many forensic investigations **do not require explicit** identification of a particular forensic trace
  
- 本文的改进

  - **open** set
  - not explicitly identify the particular forensic traces contained in an image patch, just whether they are **consistent** across two image patches

- forensic similarity system

  - feature extractor
    - use CNN to extract <u>general low-dimensional forensic features(deep features)</u> from an image patch
    - $ f : X → R^N $
      - 将图像块X映射到实数值N维特征空间
      - 特征空间encode图像块X的high-level forensic information
      - convolutional neural networks (CNNs) are powerful tools for extracting general, high-level forensic information from image patches
  - a three layer neural network
    - map pairs of these deep features onto a similarity score
    - $ S : R^N× R^N→ [0,1]$
      - 将两对forensic feature vectors映射到一个范围为0到1的相似度分数
      - 低相似度分数表明两个图像块X1和X2具有不同的取证痕迹，而高相似度分数表明两个取证痕迹高度相似
    - 最后，将两个图像块X1和X2的相似性得分$ S(f(X1), f(X2)) $与阈值η进行比较

- forensic trace 

  - source camera model / manipulation type / editing parameter
  - ①the camera model that captured the image;②the social media website where the image was downloaded from;③the processing history of the image
  - inherently unrelated to the perceptual content of the image

- **Siamese network**

### FORENSIC SIMILARITY

> motivate and formalize the concept of forensic similarity

1. 输入：图像块X1和X2
2. 一个特征提取器将两个输入映射到一对特征向量f(X1)和f(X2)，它对有关图像块的high-level forensic information进行编码
3. 相似度函数将这<u>两个特征向量(该阶段的输入)</u>映射到相似度分数，然后将其与阈值进行比较。
   - 阈值以上的相似性分数表示X1和X2具有相同的取证轨迹（例如处理历史或源相机模型）
   - 阈值以下的相似性分数表明它们具有不同的取证轨迹

![forensic similarity system](digital-images-forensic\tif19-forensic-similarity.png)

![detailed-forensic similarity system](digital-images-forensic\tif19-forensic-similarity-1.png)

### PROPOSED APPROACH

> detail proposed deep-learning system implementation and training procedure.
>
> describe how to build and train the CNN-based feature extractor and similarity network

- patch selection

### EXPERIMENTAL EVALUATION

> evaluate the effectiveness of our proposed approach in a number of forensic situations, and importantly effectiveness on unknown forensic traces

- Source Camera Model Comparison

  ![Source Camera Model Comparison](digital-images-forensic\SourceCameraModelComparison.png)

  - Patch Size and Re-Compression Effects
  - compare with other approaches
  - impact of training procedure → unfrozen / update + subset / diverse dataset
  - architecture variants
    - green channel → full color

- Editing Operation Comparison

- Editing Parameter Comparison

### PRACTICAL APPLICATIONS

> demonstrate the utility of this approach in two practical applications.

- image forgery detection and localization
  - localization：
    - 选择原图作为参考补丁，则不一致的是拼接图
    - 选择拼接图为参考补丁，则不一致的是原图，我理解为“双向定位”
- image database consistency verification
  - 看着这个数据库是不是由同一个相机生成的照片构成

### CONCLUSION

---

## 🧨Noiseprint: a CNN-based camera model fingerprint

### 运行项目

项目地址：https://github.com/grip-unina/noiseprint

jpeg压缩的质量因子要求51~101

 <img src="digital-images-forensic\cuda-error.png" alt="img" style="zoom: 50%;" />

``` python
import tensorflow.compat.v1 as tf
tf.disable_v2_behavior()
```

运行：

```bash
python main_blind.py dog1.png mat ou.jpg
python main_map2uint8.py mat.npz test.png
python main_showres.py dog1.png dog1.png mat.npz
```

python os join path的问题：https://blog.csdn.net/weixin_37895339/article/details/79185119

### Abstract

> extract <u>a camera model fingerprint → noiseprint</u>
>
> image forgery localization

noise residual 噪声残差

siamese networks 暹罗网络

### INTRODUCTION

**any acquisition device leaves on each captured image distinctive traces**

- statistical methods, based on `pixel-level` analyses of the data 基于数据的像素级分析的统计方法
  - a model-based approach：建立一些特定特征的数学模型，并将其用于法医目的
    - popular targets
      - lens aberration
      - camera response function
      - color filter array(CFA)
      - JPEG artifacts
    - 缺点：适用范围窄
  - a data-driven approach
    - noise residual
      - 概念：the noise-like signal which remains once the high-level semantic content has been removed
      - 如何获取：
        - ①从图像中减去通过降噪算法(denoising algorithms)估计的“干净”版本
        - ②通过在空间或变换（傅里叶，DCT，小波）域中应用一些高通滤波器
      - PRNU-based methods的缺点
        - the need of a large number of images taken from the camera to obtain good estimates
        - the low power of the signal of interest with respect to noise
          - 主要的噪声源是高级图像内容，由于不完善的过滤，该图像内容在PRNU中泄漏

- 通过提取noiseprint可以发现被篡改的图片中的不一致性

  ![1611413292011](digital-images-forensic\noiseprint-inconsistency.png)

> propose a new method to extract <u>a noise residual → noiseprint</u>

noiseprint shows clear traces of camera artifacts

###  RELATED WORK

> analyze related work on noise residuals to better contextualize our proposal

- Exploiting noise for image forensics 利用噪声进行图像取证
  - unsupervised methods
  - intrinsic fingerprint
- Using deep learning for image forensics
  - 缺点：rely on a training dataset strongly aligned with the test set

###  PROPOSED APPROACH

> describe the proposed architecture and its training

因为内部处理步骤，每个相机模型在每个获取的图像上留下了模型本身特有的许多**伪像(sartifacts)**，但是这些伪像很**脆弱**而且他们的利用需要**极其复杂的统计方法**，最经典的方法是借助于高通滤波器(high-pass filter)或去噪器(denoiser)提取图像的噪声残留(a noise residual)

> 1. improve the noise residual extraction process
> 2. enhance the camera model artifacts 

- Extracting noiseprints
  
  - 输入：a generic image 通用图像
  - 输出：a suitable noise residual 
  
  ![1611720504128](E:\hexo\source\_posts\digital-images-forensic\extract-noiseprint.png)
  
  - denoiser
    - 删除或强烈衰减高级场景内容，这对于我们的目的而言是一种干扰
    - 不是去生成无噪声版本，而是提取影响其的噪声模式（通过去除高级内容），最终将其从输入中减去以获得所需的清晰图像
  
    ![1611733547523](E:\hexo\source\_posts\digital-images-forensic\denoiser.png)
  
    - 训练基于CNN的降噪器。输入：a noisy image patch；输出：its noise content
  
  - 原理：**image patches coming from the same camera model should generate similar noiseprint patches**
  
- Implementation

  - initialization
  - boosting minibatch information：$O(n^2)$
  - distance-based logistic loss
  - regularization

### EXPERIMENTAL ANALYSIS

> carry out a thorough comparative performance analysis of a noiseprint-based algorithm for forgery localization

- forgery localization based on noiseprints
  - 输入：the image and its noiseprint
  - 输出：a real-valued heatmap for each pixel
  - Splicebuster：通过采用图像噪声纹代替其中使用的三阶图像残差而获得的改善效果
- reference methods
- datasets
- performance measures
- training procedure
  - training训练 / validation验证 / test测试
- preliminary analyses

### FURTHER NOISEPRINT-BASED FORENSIC ANALYSES

> provide ideas and examples on possible uses of noiseprints for further forensic tasks

### CONCLUSIONS

---



基本情况：两篇仔细阅读和整理完毕，代码找到，但未研究具体细节

# 人工智能

一个人工智能的诞生：https://jibencaozuo.com/



---

# 汇报

## 👼 FIRST

Media Forensics and DeepFakes

SECTION B通读，重点看**106(老师没看)**，**107**是如何提到的 **96(特征很复杂)**，整合自己的方案，探讨自己的问题，最后再用标准库测试

![1611749784564](E:\hexo\source\_posts\digital-images-forensic\paper.png)

blind scenario

传统的、盲方法(不需要先验知识)

基于有监督的机器学习，如SVM

聚类：kmeans(指定扩充的类别) / 自适应扩充的类别

理解工作、没有代码自己编、无需百分百重现、寻找亮点

难度和复杂度自己控制、最后要检测定位的目的

结合实际篡改取证的需求 + 精读文章 + 优化、组合、融合

假设图像来源 / 最有可能的篡改：缩放、边界融合、压缩

引用式的做对比

对比实验也很重要、子实验越多越好、越可信

注意一下三篇文献的**去噪方法**、做实验的时候看中间结果

最后的改进和创新都是从**细节**开始

两大类实验：基本库 / 稍微带点鲁棒性(对数据库整体进行后处理)

**1. 做什么 / 怎么做 2. 为什么做(整体方案/每个模块/细节参数设置)**

自己讲3篇具体算法 / 自己对为什么的理解、重点模块 / 自己的初步实验规划、系统图 / 细化 / PPT，多画图，多用图示

- 【96】 D. Cozzolino, D. Gragnaniello, and L. V erdoliva, “Image forgery localization through the fusion of camera-based, feature-based and pixel-based techniques,” in IEEE International Conference on Image
  Processing, 2014, pp. 5302–5306.
- 【106】 D. Cozzolino, F. Marra, G. Poggi, C. Sansone, and L. V erdoliva, “PRNU-based forgery localization in a blind scenario,” in International Conference on Image Analysis and Processing, 2017, pp. 569–579.
  - 见📕591页
- 【107】 L. V erdoliva, D. Cozzolino, and G. Poggi, “A feature-based approach for image tampering detection and localization,” in IEEE International Workshop on Information F orensics and Security, 2014, pp. 149–154.



---

## 🧨Image forgery localization through the fusion of camera-based, feature-based and pixel-based techniques

