---
layout: post
title: Advanced Image Registration
date: 2023-10-10 08:00:00
description: Self-learning note for advanced image registration.
tags: mia
categories: study ucph
related_posts: true
toc:
  sidebar: left
giscus_comments: true
thumbnail: assets/img/advanced-image-registration.png
---

#### Todays Learning Objectives

- What does **<u>regularization</u>** do in the context of **<u>registration</u>**?

  In the context of image registration, regularization refers to a technique used to **improve** the **accuracy** and **robustness** of the registration process. Here's **what regularization does**:

  - **Control Deformation**: Image registration aims to **<u>align two images by finding a transformation that maps one image onto the other</u>**. **<u>Regularization</u>** helps control the **<u>degree of deformation or warping allowed during this alignment</u>**. It **prevents** overly **<u>complex and unrealistic deformations</u>** that may lead to poor results.
  - **Penalize Irregularities**: Regularization introduces **<u>a penalty term</u>** in the registration objective function. This penalty term **<u>discourages irregular, non-smooth deformations</u>**. By doing so, it **<u>promotes smooth and plausible transformations</u>** between the images.
  - **Balancing Trade-Off**: Regularization **<u>strikes a balance</u>** between **<u>fitting the images together accurately</u>** and ensuring that the **<u>deformation remains physically plausible</u>**. Without regularization, registration can result in **<u>overly complex or noisy transformations</u>** that may not align images effectively.
  - **Controlling Sensitivity**: It also helps control the **<u>sensitivity</u>** of the registration process **<u>to noise or outliers</u>** in the data. Regularization can make the registration process **<u>more stable and less prone to small variations</u>** in the images.

  In summary, regularization in image registration ensures that the **<u>alignment process is not overly complex</u>** and **<u>produces physically meaningful transformations</u>** that are both accurate and stable.

- <u>Name</u> and <u>explain</u> at least two <u>clinical applications of registration</u>

  1. **Image-Guided Surgery**: Image registration is widely used in surgery. It involves **<u>aligning preoperative medical images</u>** (such as CT scans or MRI scans) with the **<u>patient's actual anatomy</u>** during surgery. This allows surgeons to **<u>visualize the patient's internal structures</u>**, **<u>plan surgical procedures more accurately</u>**, and navigate in real-time during surgery. Image-guided surgery improves **<u>precision</u>** and <u>**reduces the invasiveness of procedures**</u>.
  2. **Radiation Therapy**: In radiation therapy for **<u>cancer treatment</u>**, image registration is crucial. It involves **<u>registering a patient's diagnostic images with images taken during the treatment</u>** (such as cone-beam CT scans). This ensures that **<u>radiation beams precisely target the tumor while sparing healthy tissues</u>**. Accurate registration is essential to **<u>maximize treatment effectiveness</u>** and **<u>minimize side effects</u>**.

- Handle <u>open data sets</u>

  Handling open datasets involves **<u>working with publicly available datasets</u>** for research or educational purposes. Here's how you can handle open datasets:

  1. **Data Access**: **<u>Find open datasets relevant to your research or learning objectives</u>**. These datasets are often available through websites, data repositories, or research organizations. Ensure that you have the necessary permissions to access and use the data.
  2. **Data Download**: **<u>Download the datasets</u>** following the provided guidelines and terms of use. Some datasets may require registration or citation, so make sure to adhere to any specific requirements.
  3. **Data Preprocessing**: Depending on the dataset, you may need to **<u>preprocess the data, clean it, and format it for your specific analysis or research goals</u>**. This step might involve data cleaning, normalization, and transformation.
  4. **Data Analysis**: **<u>Perform the desired data analysis</u>**, which could include image registration, image analysis, or other tasks related to your research or learning objectives. Ensure you follow good data analysis practices and documentation.
  5. **Ethical Considerations**: **<u>Always handle open datasets with ethical considerations</u>**. Respect privacy, follow legal regulati

#### Regularization

**Terms**

**Fixed Image**:

- The "fixed" image is the **reference image** or the image that **serves as the target** for the registration process.
- It's the image **you want the "moving" image to be aligned** with or transformed to match.
- In medical image analysis, the fixed image is often an anatomical image, such as a CT scan or an MRI scan, which provides the reference structure for alignment.

**Moving Image**:

- The "moving" image is the image that **you want to align with the fixed image**.
- It's the image that **undergoes a transformation** to bring it into spatial alignment with the fixed image.
- In medical image analysis, the moving image could be another image of the same patient acquired at a different time or with a different modality.

The goal of image registration is to **find a transformation** that can **map or align** the **moving image to match the fixed image** as closely as possible. This process is commonly used in fields like medical image analysis to compare images taken at different times, with different modalities, or for other clinical or research purposes. The fixed and moving images help **establish a spatial relationship between different images, enabling comparisons, analyses, and clinical decision-making**.

**Deformation** refers to the process of **changing the <u>shape</u>, <u>size</u>, or <u>spatial arrangement</u> of an object or a structure**. In the context of image registration, deformation typically involves **<u>altering the spatial configuration of an image or a part of an image</u>** to bring it into **<u>alignment with another image</u>**. Deformation can be applied to individual pixels or voxels within an image to achieve this alignment. It's a fundamental concept in image registration because **<u>it describes how one image is modified to match the other</u>**.

**Transformation**, on the other hand, is a broader term that **<u>encompasses any mathematical operation</u>** applied to an image to change its position, orientation, scale, or shape. A transformation can be rigid (only involving translation and rotation), affine (involving translation, rotation, scaling, and shearing), or non-rigid (involving deformation). Transformation is a **<u>more general term</u>** that includes deformation as a specific case.

**Deformed Moving** in the context of image registration means that the **"moving" image, which was originally not aligned with the "fixed" image**, has undergone a deformation or transformation to achieve spatial alignment with the fixed image. This term indicates that **the moving image has been modified or adjusted to fit or match the spatial characteristics of the fixed image**. Deforming the moving image is a key step in the registration process, where the transformation is applied to bring the two images into spatial concordance.

Here we have an example,

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/0MY6HKQ.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 1. Fixed (reference) and moving (ongoing) images
</div>

Here is the **deformed moving** image.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/QQqviCR.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 2. Deformed moving image
</div>

**_Is this a good transformation field?_**

**Definitely not**. There are a lot of **strange internal distortions**. The transformation field might not be ideal or that the registration process did not achieve the desired result.

**_Solution_**

**Regularization**

- We make the algorithm **pay the price** for **too much deformations**
- **Price** should **grow nonlinearly** (better **move many a little bit** than **one a lot**)

This phrase is explaining the **rationale behind the nonlinear growth of the cost**. It suggests that it's **more desirable to apply small deformations to multiple regions** of an image rather than **applying a very significant deformation to just one region**. This is often because **small deformations are more likely to preserve the overall structure and quality** of the image.

To put it simply, in image registration with regularization, the **cost associated with deformation should encourage small, gradual changes across the entire image** rather than allowing or favoring large, abrupt changes in just one area. The idea is to **ensure that the transformation or deformation is smooth and physically meaningful**, which can lead to **better registration results** and **more realistic alignments**. This approach helps **prevent unrealistic distortions or artifacts** in the transformed image.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/VQqGWET.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 3. Definition of points
</div>

The idea behind is that **we would like to preserve distances between**:

· Point $i, j$

· Point from $\Omega_{i, j}$

And we would like to **<u>preserve distances between points</u>** in original and deformed grid

The goal of "**preserving distances between points**" is to ensure that, **after the deformation**, the **<u>distances or spatial relationships between points on the deformed grid remain as close as possible to the distances between the corresponding points on the original grid</u>**. In other words:

- If **<u>point A is originally a certain distance away from point B on the original grid</u>**, we want to ensure that **<u>after deformation</u>**, **<u>point A remains a similar distance away from point B on the deformed grid</u>**.
- This preservation of distances helps **<u>maintain the local structure and spatial relationships in the image</u>**. It ensures that important **<u>anatomical or structural features</u>** in the image **<u>do not get distorted or altered</u>** significantly during the registration process.

Overall, this concept is fundamental in regularization during image registration. It contributes to the smoothness and physical plausibility of the deformation, preventing unrealistic warping and ensuring that the registered image accurately reflects the spatial characteristics of the original image.

**Problem formulation**

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/P3XM5Gr.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 4. Formulation of problem
</div>

**Original optimization formulation:**

$$
F(I, J, \theta) = \min\sum_{i, j}d\left(I(i, j) - J(x(i,j,\theta), y(i, j, \theta))\right)
$$

**Formulation with regularization:**

$$
F(I, J, \theta) = \min\left(\sum_{i, j}d(I(i, j) - J(x(i, j, \theta), y(i, j, \theta))\right) \\
+ \lambda\sum_{i,j}\sum_{k, l\in\Omega_{i, j}}\left(((i-k)\ - (x(i, j, \theta) - x(k, l, \theta))\right)^2
$$

We add a regularization term to **give a penalty**. And what will this regularization do?

$$
\sum_{i, j} \sum_{k.l\in\Omega_{i, j}}(\left(1-sign\left((i-k)\cdot(x(i, j, \theta) - x(k, l, \theta))\right)\right) + \left(1-sign\left((j-l)\cdot(y(i, j, \theta)-y(k, l, \theta))\right)\right))
$$

This regularization term is used to **<u>encourage the preservation of distances or relationships</u>** between neighboring points in the image grid during the registration process. It **<u>penalizes situations</u>** where the **<u>distances between points change significantly</u>** as a result of the deformation caused by the transformation parameter $\theta$.

The term 1−sign(…) essentially **<u>imposes a penalty</u>** when the transformation causes **<u>significant changes in the spatial relationships between points</u>**. This encourages the registration process to **<u>produce a deformation that maintains the local structure and spatial characteristics of the image</u>**.

The "folding" you mentioned likely refers to the effect of this regularization term in **<u>preventing excessive deformations that could distort or fold the image</u>**. The regularization term **<u>discourages such deformations</u>**, promoting smoother and more physically plausible transformations.

In summary, this regularization term helps **<u>maintain the spatial relationships between neighboring points in the image grid</u>**, preventing excessive distortion during image registration.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/LbWQENU.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 5. Regularization of this term
</div>

And here, by using regularization, we get a better deformed result.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/tiWJyvY.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 6. Regularized result
</div>

#### Landmarks

Can we help registration of two squares?

- Automatically detect square corners

  This step involves the automatic identification of corners or key points on the square objects in the images. These corners are distinctive features that can be used as landmarks for registration.

- Force registration to align corners

  - This means using the detected square corners as constraints during the registration process. The registration algorithm will be instructed to ensure that the corners of the squares align precisely or as closely as possible.
  - By "forcing" the registration to align corners, you are making it a priority to match these specific, distinctive points. This can be especially useful when you have prior knowledge of where these corners should be located in the registered image.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/fb1LYL8.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 7. Landmarks
</div>

Then we have new optimization function.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/UzjYqzQ.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 8. New optimization function
</div>

The use of landmarks is a common technique in image registration, and it's particularly valuable when you have objects with distinctive and easily detectable features, such as square corners. By aligning these landmarks, you can improve the registration accuracy and ensure that specific points of interest are correctly positioned in the registered image.

**How to ensure the landmarks match?** Validate the results by checking how closely the landmarks in the registered images match. You can calculate and evaluate the distances between corresponding landmarks to assess the quality of registration.

#### Summary Algorithm

- **Define algorithm settings**
  - Acceptable transformations (Rigid, Affine, Non-rigid)
  - Similarity measure (Mean Squares, Correlation, Mutual Information)
  - Multi-resolution pyramid, grid schedule
  - Regularization
- **Prepare input images**

  - Normalization of intensities
  - Rescaling

- **Optimization**
  - Compute similarity measure over **grids on both images**
  - Compute **gradients on grid** on moving image
  - Move **grid according to the gradients**
  - **Deform** moving image according to the **grid**
  - Use splines to **interpolate** pixels outside the grid
- Repeat until convergence

#### Applications

##### Atlas-based segmentation

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/haDNA69.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 9. Atlas-based segmentation
</div>

- Advantages:

  - Anatomically correct segmentation

    这意味着使用基于参考图谱的分割方法可以获得解剖学上正确的分割结果。具体来说，该方法参考了已知的解剖结构信息，因此分割结果更有可能准确地匹配到具体的解剖结构，如器官、组织或区域。这有助于提高分割的准确性，尤其是在医学图像分析中，这对于诊断和研究非常重要。

  - Light training image requirements

    与某些其他分割方法相比，基于参考图谱的分割通常需要较少的训练图像。这意味着你不必拥有大量的标记数据来训练分割模型。相反，你可以仅使用一个或几个高质量的参考图谱，然后将其应用于其他图像，从而降低了训练数据的需求和数据标记的工作量。

  - Invariant to the number of target objects

    这意味着这种方法的性能不受需要分割的目标对象数量的影响。你可以使用相同的参考图谱和算法来分割不同数量的目标对象，而不需要根据目标数量进行调整或重新训练。这种不变性对于处理不同图像或场景中的多个目标对象非常有用，因为它减轻了定制和调整的工作。

- Disadvantages:

  - Slow

    **计算复杂性**：基于参考图谱的分割方法通常涉及计算图像之间的配准或对准。这包括计算变换场（deformation field）以将参考图谱与目标图像对齐。这些计算在像素级别进行，特别是在非刚性（non-rigid）变换的情况下，需要大量计算。这会导致算法的复杂性和计算时间的增加。

    **多分辨率金字塔**：虽然多分辨率金字塔可以提高配准的鲁棒性，但也会增加计算的复杂性。多分辨率金字塔需要在不同分辨率级别上执行配准，每个级别都需要计算变换场。这意味着多分辨率金字塔方法可能需要更多的计算时间。

    **正则化**：正则化用于平滑变换场以确保物理上合理的形变，但它也会引入额外的计算。正则化通常需要迭代优化过程，这会增加计算时间。

    **硬件限制**：图像分割通常需要大量计算资源，特别是在高分辨率或大规模数据集的情况下。如果计算资源受限，速度可能会受到限制。

See details about [Atlas-based Segmentation](https://liuying-1.github.io/blog/2023/image-registration-l1/) :point_left:

##### Multi-modal image registration

Improves **visibility of structures** (bones – CT, soft tissues – MR, tumors – PET and MR)

"Multi-modal image registration" 指的是在医学图像分析中，同时注册或配准不同模态（多模态）的医学图像。**<u>每种模态</u>**的医学图像（如CT扫描、MRI扫描、PET扫描等）**<u>对人体结构和病变有不同的灰度值、对比度和信息</u>**。"Improves visibility of structures" 的意思是**<u>多模态图像配准可以改善不同模态下的结构可视性</u>**。

具体来说，这些不同模态的图像可能有以下特点：

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/SBq4Pxv.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 10. Soft-tissues
</div>

1. **CT图像**：在CT扫描中，**<u>骨骼结构</u>**通常显示得很清晰，因为它们有高对比度。然而，对软组织的可视性可能较差。

2. **MR图像**：MRI图像对于显示**<u>软组织结构</u>**非常出色，但对于骨骼结构的可视性可能不如CT。

   <div align=center>
   <div class="row mt-3">
       <div class="col-sm mt-3 mt-md-0">
           <img src="https://i.imgur.com/HKTrqHQ.png" class="img-fluid rounded z-depth-1" data-zoomable />
       </div>
   </div>
   </div>
   <div class="caption">
     Figure 11. Bones
   </div>

3. **PET图像**：PET扫描用于检测代谢活跃性，通常用于**<u>肿瘤诊断</u>**。PET图像可以显示**<u>肿瘤</u>**和其他异常的生物活动，但对解剖结构的细节可视性相对较低。

   <div align=center>
   <div class="row mt-3">
       <div class="col-sm mt-3 mt-md-0">
           <img src="https://i.imgur.com/VM8awXw.png" class="img-fluid rounded z-depth-1" data-zoomable />
       </div>
   </div>
   </div>
   <div class="caption">
     Figure 12. Tumors
   </div>

"Multi-modal image registration" 允许将这些不同模态的图像进行配准，以便在同一坐标系下对比它们。这样做有以下好处：

- **改善可视性**：通过将不同模态的图像进行配准，你可以在一个图像中同时显示不同模态下的结构。例如，你可以在同一图像中清晰地看到骨骼（来自CT图像）、软组织（来自MR图像）和肿瘤活性（来自PET图像），从而获得更全面的信息。
- **指导诊断和治疗**：多模态配准可帮助医生更准确地诊断病症、规划手术和监测治疗进展。医生可以在一个图像上同时查看多个信息来源，有助于更全面地了解患者的情况。

总之，多模态图像配准有助于提高医学图像的可视性，增加信息的融合，从而更好地支持医疗诊断和治疗过程。

多模态图像配准是将来自不同图像模态的图像进行对齐，以使它们在同一坐标系下对应到相同的解剖结构或区域。这种配准的目的是允许医生或研究人员同时查看不同模态下的图像信息，从而更全面地理解患者的情况。以下是多模态图像配准的一般流程：

1. **图像获取**：首先，不同模态的图像（例如CT、MRI、PET等）必须分别获取。这些图像通常在不同的设备上获得，具有不同的对比度和信息。
2. **图像预处理**：在进行配准之前，通常需要对图像进行预处理，包括去噪、增强、图像校正等操作，以确保它们处于最佳状态。
3. **特征提取**：从每种模态的图像中**<u>提取特征</u>**，这些特征可以是解剖结构、标记点或其他在不同模态下容易识别的图像特征。特征提取是关键的一步，因为**<u>它确定了配准过程中用于对齐的关键点</u>**。
4. **特征匹配**：在不同模态图像中提取的特征进行匹配。这可以通过各种计算机视觉算法实现，通常是在配准算法中的一个步骤。特征匹配的目标是找到相应的特征点，以便进行配准。
5. **配准变换**：**<u>基于找到的特征匹配，计算需要应用于其中一个图像以使其与另一个图像对齐的变换</u>**。这可以是刚性、仿射或非刚性变换，具体取决于图像的类型和所需的对齐程度。
6. **配准优化**：进行迭代优化，以确保变换能够最好地对齐图像。这包括优化变换参数以最小化特征点之间的距离或匹配误差。
7. **生成配准后的图像**：应用配准变换，将不同模态的图像对齐到同一坐标系下，从而生成配准后的图像。这些图像可以同时查看，以获得更多信息。

需要注意的是，多模态图像配准是一项复杂的任务，其复杂性取决于图像的特性和所使用的配准算法。它通常需要计算机视觉和图像处理领域的专业知识，以确保高质量的配准结果。这种配准方法在医学影像学、医学诊断和研究中具有广泛的应用。

##### Atlas-based abnormality detection

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/wFbSCgB.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 13. Abnormality detection
</div>

- Register **<u>healthy</u>** image to **<u>potentially abnormal</u>**
- Study deformation field to **<u>find abnormalities</u>**

"Atlas-based abnormality detection" 是一种用于检测医学图像中异常或病变的方法，它基于已知的正常解剖结构的图谱（atlas）来识别图像中的异常情况。

1. **图谱（Atlas）**：图谱是一组已知正常解剖结构的参考图像。这些图像通常来自大量的**<u>健康患者</u>**，用于表示**<u>正常的生理结构</u>**和器官。图谱可以包括各种模态的图像，如CT、MRI等。
2. **异常检测**：在医学图像中，异常通常是指与正常解剖结构不同的区域，可能是疾病、损伤或其他异常情况的迹象。异常可以表现为异常的形状、密度、亮度或其他特征。
3. **配准与对比**：在进行异常检测时，首先将**<u>待检测图像（可能是患者的图像）与正常图谱</u>**进行配准。这意味着将待检测图像中的结构与图谱中的对应结构对齐。
4. **特征提取**：一旦完成配准，就可以从待检测图像中提取特征，这些特征通常是与异常相关的特征，如区域的形状、密度、纹理等。
5. **异常检测算法**：使用特征和已知的正常图谱作为参考，异常检测算法可以**<u>识别待检测图像中与正常图谱不匹配</u>**的区域。这些不匹配的区域可能表示异常或病变。
6. **可视化和报告**：检测到的异常区域可以用于生成可视化结果，供医生或研究人员查看。异常检测的结果通常以可视化图像或报告的形式呈现。

"Atlas-based abnormality detection" 的优势在于它依赖于已知的正常图谱作为参考，因此可以帮助**<u>快速识别潜在的异常区域</u>**。这对于医学诊断、疾病筛查和研究非常有用，因为它可以帮助医生或研究人员更快速地识别异常，而不需要手动检查整个图像。

在 "Atlas-based abnormality detection" 中，通常采取的方法包括：

1. **将健康图像配准到潜在异常图像**：首先，将一个或多个健康图像（正常图谱）与潜在异常图像进行配准，以使它们在相同的坐标系下对齐。这意味着**<u>将健康图像中的正常结构与潜在异常图像对应的结构对齐</u>**。
2. **研究变换场以发现异常**：**<u>一旦进行了配准，就可以研究变换场</u>**（deformation field）。变换场表示**<u>从健康图像到潜在异常图像的变换</u>**，它描述了**<u>正常结构在潜在异常图像中的形变</u>**。通过**<u>分析变换场</u>**，可以**<u>识别出在潜在异常图像中与正常结构不匹配</u>**的区域。

这个方法的思想是，通过将健康图像与潜在异常图像对齐，可以将正常结构的位置和形状信息传递到潜在异常图像中。然后，通过比较正常结构的期望位置和实际位置（通过变换场表示），可以检测到异常或病变区域，因为这些区域可能会产生异常的形变。

这种方法在医学图像中用于异常检测，特别是在需要自动识别潜在异常或病变的情况下，具有重要的应用潜力。

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/itVoHD4.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 14. Small abnormalities
</div>

- Problem
  - Difficult to detect small abnormalities

"Difficult to detect small abnormalities" 意味着在某些情况下，使用基于图谱的方法来检测小尺寸异常或病变可能面临挑战。这可能是因为以下原因：

1. **空间分辨率限制**：医学图像的空间分辨率是有限的，特别是对于某些成本昂贵的成像技术。小尺寸的异常可能在图像中占据的像素数量很少，因此可能难以在低分辨率图像中准确检测。
2. **噪声和伪影**：医学图像可能受到噪声和伪影的影响，这些因素可以干扰小异常的检测。噪声可以导致虚假的异常检测，而伪影可能使异常区域难以分辨。
3. **特征提取的挑战**：检测小异常通常需要更高级别的特征提取和分析，以便捕捉细微的变化。这可能需要更复杂的算法和更多的计算资源。

解决这个问题的方法可能包括：

- **提高图像分辨率**：使用更高分辨率的成像技术可以帮助更准确地检测小尺寸的异常。
- **降低噪声**：采取噪声抑制技术，以减少噪声对异常检测的干扰。
- **使用多模态信息**：结合不同模态的图像信息，以提高对小异常的检测能力。
- **使用深度学习方法**：深度学习技术在医学图像分析中表现出色，可以帮助检测小异常，因为它们可以自动学习图像中的特征。

综合来说，检测小异常是医学图像分析中的一个挑战，但可以通过改进图像质量、采用高级分析方法以及使用多种信息源来应对这一挑战。

##### Super-resolution images

We have three orthogonal 3D MR images of high in-plane resolution but high slice thickness. How to create a super-resolution 3D MR?

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/AfM6SVk.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 15. Super-resolution
</div>

创建超分辨率 3D MR 图像通常涉及将现有的低分辨率图像增强到高分辨率图像的过程。在这种情况下，你拥有三个不同方向（x、y、z）的 3D MR 图像，这些图像具有高平面分辨率，但在切片（z轴方向）上的分辨率较低。下面是一种可能的方法来创建超分辨率 3D MR 图像：

1. **数据预处理**：首先，对你的三个 3D MR 图像进行数据预处理。这包括去噪、伪影去除、校正以及确保它们已经在相同的坐标系下对齐。
2. **切片插值**：在 3D MR 图像的切片方向上，由于分辨率较低，你可以使用插值方法来增加切片数量，从而提高 z 轴分辨率。这可以采用一维插值方法，例如线性插值或样条插值。这将生成一系列新的切片，以增加在 z 轴方向上的分辨率。
3. **3D 重建**：一旦你获得了高分辨率切片，可以将它们组合成 3D 数据卷。这可以通过采用体绘图（Volume Rendering）或通过堆叠这些切片来实现。这将生成一个具有更高分辨率的 3D MR 图像。
4. **超分辨率技术**：如果你希望进一步增加图像的分辨率，可以使用超分辨率技术。这些技术基于统计方法、深度学习或其他算法，可以通过分析图像的细节来生成更高分辨率的版本。这需要具体的超分辨率算法，并可能需要使用额外的训练数据。
5. **评估和验证**：最后，一定要评估和验证生成的超分辨率图像的质量。这可以通过与高分辨率图像进行比较，或者使用图像质量指标来进行评估。

需要注意的是，超分辨率图像的质量取决于数据的质量、使用的插值和重建方法，以及是否使用了超分辨率算法。这是一个复杂的过程，通常需要在医学影像领域具有专业知识的人员来执行。

- We convert orthogonal images to full volume but adding empty space
- This empty space should not be taken into account during registration
- Idea of masked registration

在创建超分辨率 3D MR 图像时，将正交图像转换为完整的体积并添加空白空间是一个常见的方法。这种空白空间不应该在图像配准过程中考虑在内，因为它仅是为了在 z 轴方向上增加分辨率而添加的。

以下是关于如何执行这一过程的一般步骤：

1. **创建完整的 3D MR 体积**：将正交图像（高分辨率但切片分辨率较低）转换为完整的 3D MR 体积。这可以通过在切片方向上重复图像来实现，以填充空白空间。这将生成一个完整的 3D 数据卷，包括原始图像和添加的空白切片。
2. **生成掩模（Mask）**：为了确保在图像配准期间不考虑空白切片，你可以生成一个掩模，其中空白切片被标记为不感兴趣区域。掩模通常是一个与图像体积具有相同尺寸的 3D 数据，其中正值表示感兴趣区域，负值或零表示不感兴趣区域（即空白切片）。
3. **掩模配准**：在进行图像配准时，你可以使用掩模来指定感兴趣区域。这意味着只有非空白部分的图像将被用于配准，而空白部分将被忽略。这有助于提高配准的准确性，因为配准算法将集中在包含有用信息的部分。
4. **生成高分辨率 3D 图像**：一旦完成配准，可以生成具有更高 z 轴分辨率的 3D MR 图像。这可以通过将正交图像的 z 轴方向上的高分辨率信息与掩模应用于已配准的体积来实现。
5. **评估和验证**：最后，一定要评估和验证生成的超分辨率 3D 图像的质量，以确保它满足你的需求。

这种方法允许你在 z 轴方向上增加分辨率，同时保留了 x 和 y 轴方向上的高分辨率信息。使用掩模来指定感兴趣区域是一个有用的技巧，以确保只有有用的信息用于图像配准和超分辨率生成。

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/GoKFaDW.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
</div>
<div class="caption">
  Figure 16. Super-resolution - 2
</div>

I didn't get the points here, and there are some other applications afterwards, but I am not going to explore them in detail, for instance, **Adaptive radiotherapy planning**, **Landmark-based registration**, and **2D-3D registration**.
