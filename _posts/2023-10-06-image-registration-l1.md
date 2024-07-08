---
layout: post
title:  Image Registration Basics
date: 2023-10-06 08:00:00
description: Learning note for medical image registration.
tags: mia
categories: study ucph
related_posts: true
toc: 
  sidebar: left
giscus_comments: true
thumbnail: assets/img/image-registration.png
---

***Disclaimer: All notes below are refered to the course "Medical Image Analysis" delivered by UCPH.***

So today's lecture will be on **<u>Image Registration</u>**.

We have already talked about the topic of registration in our previous lecture. I said that there is some problems of registration. And its idea is to move from one image to another. It;s very popular in our field but I initially said we're not talk about applications of registration so that you can think about this yourself, and also we can wait till the registration lecture like <u>which is having now</u>.

#### Todays Learning Objectives

- Why do we need registration?
- Name and explain at least two similarity measures
- Describe the difference between rigid, affine and non-rigid registration

####  What is Registration?

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/i4jYCLQ.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 1. Example of registration
</div>


- <u>Image processing method</u> for <u>aligning two or more images with each other spatially</u> (geometrically)
- You can <u>make images from different imaging modalities, times, sections, angles</u>, etc, <u>comparable</u>.

#### Why do we need registration?

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/jO6tT6z.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 2. Why we need registration
</div>

***Registration geometrically transforms one image into another.***

So what we have is we have an image which is in the most top left. And we have a target which is in the most right bottom.

And we slowly transform our image step by step towards the target.

So you can see that this patient has some degenerative disease and changing in the brain, so the ventricles are growing, and you can see they slowly step by step grow when we use the image of the patient. Grow and grow until they reach this level.

But the question is, how does that happen? What mathematical process lies behind allows us to nicely deform one image towards another one?

**<u>Before we go into mathematical concepts, let's look at one of the main applications of image registrations.</u>**

##### Atlas-based segmentation

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/BJ0T49b.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 3. Atlas registration
</div>

It's **atlas-based segmentation**.

Before deep learning, that was specification was the main like every time when people use registration almost in 80% or something. They used it for atlas-based registration.

So what is the idea of atlas-based registration? What we do is we have a collection of images <u>maunally segmented</u>. And we get a new image, we **<u>register every image from our collection towards our image</u>**. During this registration, we get **transformation matrix**.

So basically we get some map which says which pixels went where during the registration, how we changed during the process. And the way you can use this transformation metrics or we can apply it to the segmentation from our atlas.

我的理解是我们原本有一系列标准的人工分割好的图片，将每一张图片都以我们的新图像作为 Target 进行配准，在配准过程中，我们获得了变换矩阵，而这些矩阵可以应用到分割上从而获得新图像的 segmentation。

We deform this image towards our target and then, we use the transformation to deform the segmentation towards our target, so that our segmentation kind of stretches and changes that tries to capture our new image. 

We can do it for all images in our atlas and then we get many versions of segmentations. And then we average them somehow (label fusion), and we just take this average result as a registered segmentation result.

This is one of the main idea for using registration.

What are the **advantages** and **disadvantages** of atlas-based segmentation?

- Segmentation speed linearly depends on the number of training images $\implies adv$

  For any other algorithms, like deep learning algorithms, the number of training images does not affect the speed of segmentation itself. It can affect training but does not affect the segmentation. If we have million training images, the speed is the same.

  Because we registered every atlas image individually to our target, the speed directly linearly increases with the number of training images. $\implies disadv$ So the people investigate how we can use not all the training images but also some specific training images which are most similar to our target. So they apply some preprocessing and figure out which training images are most similar and then only register them to our target.

- Segmentation speed does not depend on the number of target structures

  There's a couple of **advantages**. One advantage is that the **speed of atlas-based segmentation** **does not depend on the number of structures we have**. We can have one million different regions in the object but the speed will be the same. We just take training images, register them, take the transformation metrics and then deform the mask.

  But many alternative algorithms, maybe like random walker, their **speed grows linearly with the number of objects we have**.

- Needs way less training samples than deep learning

  Another advantage is that we need way less training images for algorithm to work.

  In contrast to now deep learning, we need thousands or hundreds images, we can use 20 segmented brains to have a reasonably good atlas-based segmentation.

##### Detecting differences

**Diffcult to see and examine differences/changes** if the images are **not oriented** the same or **on top of each other**.

There are many different reasons for wanting to register images:

- <u>Get different contrasts</u> from different modalities superimposed

  This refers to the ability to <u>overlay or combine</u> <u>images obtained from different imaging modalities</u>, such as MRI, CT, or PET. <u>Each modality provides unique information or contrast about the same anatomical region</u>. <u>Registering</u> these images allows you to see <u>the anatomical structures in relation to each other</u>, providing a <u>more comprehensive view</u> of the patient's condition.

  (从不同模式的叠加中获得不同的对比)

  - Get an image with poor resolution or with little anatomical detail superimposed on an image with good anatomical information

    Sometimes, medical images may have <u>low resolution</u> or <u>lack fine anatomical details</u>. <u>Registering</u> such images with <u>higher-resolution</u> or <u>more detailed images</u> can help in <u>better visualization</u> and <u>understanding of the region of interest</u>. This process essentially <u>enhances the quality of the lower-quality image by aligning it with a reference image with better anatomical information</u>.

    (将分辨率低或解剖细节少的图像叠加到解剖信息丰富的图像上)

- Inverstigate <u>changes over time</u> (e.g., before-after treatment)

  Image registration is used to <u>compare images acquired at different time points</u>, such as <u>before and after medical treatment</u> or surgery. By <u>aligning</u> these images <u>accurately</u>, you can <u>precisely quantify changes in size</u>, <u>shape</u>, or <u>intensity</u> of structures over time. This is valuable for <u>monitoring disease progression</u> or the <u>effectiveness of a treatment</u>.

  - See if there are differences in <u>size, shape</u> or <u>intensity</u> over time - more sensitively

    Image registration helps in <u>detecting subtle changes</u> in the size, shape, or intensity of anatomical structures over time. It <u>enhances the sensitivity of your analysis by ensuring that the images are aligned correctly</u>, allowing for <u>precise measurement of changes that might otherwise go unnoticed</u>.

- To compare different individuals

  Image registration allows you to <u>align and compare images from different individuals</u>, enabling a quantitative and objective assessment of anatomical structures, pathology, or physiological variations <u>across a population</u>. This is especially valuable in research studies and clinical trials.

  - Analyze the images to see <u>differences between individuals</u>

    Image registration facilitates the <u>automated analysis of images</u> to <u>identify and quantify differences between individuals</u>. This can include <u>variations in organ size, shape, location, or pathology</u>. <u>Automated analysis</u> saves time and reduces the potential for subjective bias that may occur in <u>manual comparisons</u>.

  - Can <u>save a lot of time</u> - for example, rather than having to <u>manually draw the structures and compare them</u>

    Image registration <u>significantly reduces the need for time-consuming manual tasks</u>, such as <u>manually outlining or drawing structures for comparison</u>. By automating the alignment and analysis of images, it streamlines the research and diagnostic process, making it <u>more efficient and less prone to human error</u>.

  - <u>Register to atals</u>, and <u>extract specific region</u>

    Registering images to a common anatomical atlas allows for <u>precise localization and extraction of specific anatomical regions or structures</u>. This is <u>useful for conducting region-specific analyses</u> or for <u>guiding surgical procedures with high precision</u>. It also aids in standardizing data across different individuals for research purposes.

- Important to check that the <u>registrations are correct</u> - be sure to <u>compare the same anatomical structure</u>.

#### Ingredients of image registration

The registration algorithms have different aspects which they work with. 

One aspect is information type. What information they use are **intrinsic** or **extrinsic** information. Another one thing which actually depends on information type is the **similarity measure**. How to understand that images become more similar or less similar during the registration. And another thing is **transformation**. So for some applications that have **rigid** transformations, only translation, rotation for example. Or for some application and most often we need **non-rigid**. So we need to somehow change the relative position of pixels, how expansive areas fix some other areas to get the new object.

- <u>What type of information</u> do we utilise?
- <u>How do we quantify</u> similarity?
- <u>What kind of transformations</u> can we use?
- <u>What kind of interpolations</u> can we use?

##### Sources of information

- Intrinsic information

  - The images are visually similar (non-necessarily by absolute intensities):

    <div class="row mt-3">
        <div class="col-sm mt-3 mt-md-0">
            <img src="https://i.imgur.com/oEvhMvl.png" class="img-fluid rounded z-depth-1" data-zoomable />
        </div>
    </div>
    <div class="caption">
      Figure 4. Intrinsic information
    </div>

The intrinsic information is used when these images are visually similar to each other. The intensities are similar most important. Fro example, here you can see two same images and there is a histogram how good they match to each other. And now this image will be rotated and this histogram is moving and it's becoming more like a straight line.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/9eJMBdI.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 5. Illustration of Histogram
</div>

The histogram pixel reflects how much these two pixels are similar to each other. And the higher color is the more similar. So if you take the next pixel, how similar is the first pixel to second pixel and so on. And when we go through all pixels of first row, we compare the first pixel the first image to all pixels in the second image. And we just measure how similar they are. 

So if these images are very similar to each other, what we would expect is to see somehow the high values on the diagonal when we compare the corresponding pixels, and to see a low values outside the diagnoal, and we don't compare the corresponding pixels.

对角线上的值越大，说明两幅图片像素值视觉上越相近。

So this is the idea of the histogram.

**Extrinsic information**

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/wDl0rkq.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 6. Example of extrinsic information
</div>

- The images are too different from each other visually
- We need to **help registration by providing** **correspondences**

The extinsic information, we usually work with images which are not similar to each other. So you can see these two images, one of them is CT and one of them is PET. And the intensities here are completely different. 

For example, we can see that the mandible is more or less visible, but in PET, the mandiable is not visible at all and the bone is invisible. But tumor is very much visible at PET, very poor visible in CT.

So in that case, if the images have two different visually from each other. What we need to do is we need to provide some additional correspondences. And one of the most common thing to do is automated landmarks.

We train algorithm to the specific points on both images, and we use these specific points to align the images. So we detect these points individually on PET, and detect these points on CT individually. And then we use these points to alignt he images to non-register form. So there's a point measure to each other.

#### Different types of transformations

- Rigid
- Affine
- Non-rigid

##### Rigid and Affine transformations

See details in my previous blog post from [Signal and Image Processing - Transformations](https://liuying-1.github.io/assets/pdf/geometry.pdf). :point_left:

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/UqtiM6p.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


##### Non-rigid transformations

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/aHW956w.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


Now let's define this **mathematically**.

How is a 2D **transformation or rotation** defined mathematically?

Can we extend this to 3D?

##### Rigid registration - mathematically

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/cAO4SD8.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


<u>In what kind of applications to we want to use rigid registrations?</u>

E.g., **Intra-subject rigid body**

**Intra-Subject:** "Intra" means **within**, and "subject" typically **refers to an individual or a patient**. Intra-subject, in this context, means that **you are dealing with multiple images of the same individual or patient**. These **images are acquired at different time points** or **using different imaging modalities** while **focusing on the same anatomical region** within the **same person**.

**Rigid Body**: In the context of image registration, a rigid body transformation is **a mathematical transformation** that includes **translation**, **rotation**, and **scaling** but does **not allow for any deformation or stretching** of the image. In other words, the **relative positions and orientations of structures within the image can change**, but the **shapes and sizes of the structures remain the same**.

**Intra-Subject Rigid Body Registration**:

- This refers to the process of **aligning** and **registering** images of the **same individual or patient** that have been obtained under **different conditions** or at **different times**.
- For example, if **a patient undergoes a series of medical imaging scans over time** (e.g., MRI or CT scans) to monitor a condition or treatment **progress**, you may use **intra-subject rigid body registration to align these images**.
- The purpose is to ensure that **corresponding anatomical structures in each image are in the same spatial position and orientation**, even if the patient's position or the imaging conditions have changed slightly between scans.
- The primary goal is typically to **track changes in the patient's anatomy**, such as tumor growth, organ movement, or any other anatomical variations, with precision and accuracy.

Applications of **intra-subject rigid body registration** can be found in various medical fields, including **radiation therapy**, where it's crucial to **ensure that the radiation beam is accurately targeted at the same area during multiple treatment sessions**, or in **longitudinal studies to monitor disease progression** or **response to treatment within the same patient**.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/JpzfBT9.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


In summary, **intra-subject rigid body registration** involves **aligning and comparing images of the same individual taken at different times** or **under different conditions**. It's a fundamental technique in medical image analysis used for **monitoring, diagnosis, and treatment planning**, where precise **spatial alignment of anatomical structures** is essential.

##### Affine registration - mathematcically

However, basically, the scaling is considered as affine transformation.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/CRsio4B.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


In what kind of applications to we want to use affine registrations?

E.g. **inter-subject**

**Inter-Subject**

- "Inter" means **between or among**, and "subject" typically refers to **different individuals** or **patients** in the context of medical imaging.
- Therefore, "inter-subject" means that **you are dealing with images acquired from different individuals or patients**.

**Affine Transformation**:

- An affine transformation is a type of transformation that **includes translation, rotation, scaling**, and **shearing** but does not allow for **deformation or non-linear warping of the image**. Essentially, it's a more flexible transformation than rigid body transformations (which only include translation and rotation) but still **preserves the basic shape of objects**.

**Inter-Subject Affine Transformation**:

- This refers to the process of **aligning and registering images acquired from different individuals or patient**s.
- In medical imaging, you might have a set of images from different patients, such as MRI scans of different brains, CT scans of different chests, or X-rays of different limbs.
- Inter-subject affine transformation allows you to **align and compare these images**, despite variations in size, orientation, and scaling between individuals.
- The primary goal is to **establish a common spatial coordinate system that enables meaningful comparisons**, such as in population-based studies, group analyses, or template-based approaches.

Applications of **inter-subject affine transformation** can be found in various areas of medical research and clinical practice, such as **comparing anatomical structures across a population**, **creating population-based atlases or templates**, and **performing group-level statistical analyses**. It's particularly useful when you need to make comparisons or perform analyses that involve images from **different individuals** while accounting for variations in size, orientation, and scaling.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/ngmeDyx.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


In summary, inter-subject affine transformation involves **aligning and registering images from different individuals or patients**, allowing for **meaningful comparisons and analyses across a population**. This technique plays a crucial role in various medical image analysis applications, particularly in group studies and population-based research.

##### Example of affine registration across several subjects

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/XNxsaN5.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


##### Non-rigid registration - mathematically

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/Vw4yIRX.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


In what kind of applications to we want to use non-rigid registrations?

E.g. **intra or inter-subject non-rigid body**

A 3x3 matrix that represents **a non-rigid transformation**. **Each element in the matrix affects the deformation** of the image in different ways. The exact values of these elements determine **how the image is deformed locally**. Here's a brief explanation of the elements:

- `m1`, `m5`, and `m9` control **scaling** factors in the x, y, and z directions, respectively.
- `m2`, `m4`, `m6`, and `m8` control **shearing** or skewing effects.
- `m3`, `m7`, and `m9` control the **actual deformation or warping**.

**1. Why Use Non-Rigid Transformations?**

- **Non-rigid transformations** are used when you need to account for **more complex changes in the shape and structure of objects or regions within the image**. Rigid and affine transformations are too restrictive for scenarios where local deformations occur, such as changes in the shape of organs or tissues, as in medical image analysis.

**2. Applications of Non-Rigid Registrations (Intra or Inter-Subject):**

- **Intra-Subject Non-Rigid Registration**: In this case, you are dealing with images of the **same individual or patient at different time points**. **Non-rigid registration** can be used to account for **anatomical changes over time**, such as organ deformation during respiration, muscle contractions, or gradual changes in tissue structures due to disease progression.
- **Inter-Subject Non-Rigid Registration**: When you have images from **different individuals (inter-subject)**, **non-rigid registration** becomes valuable for **aligning images that exhibit significant anatomical variations**. Examples include aligning MRI scans of different brains, which have **varying shapes and sizes**, or comparing images of different patients with congenital or acquired deformities.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/JPahdsm.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


In both **intra-subject** and **inter-subject** scenarios, **non-rigid registration allows you to account for the complex and local variations in anatomy**, enabling **accurate comparisons, analysis, and medical interventions**. It's particularly important when dealing with deformable structures and when rigid or affine transformations are insufficient to capture the necessary deformations. Applications range from disease monitoring to surgical planning and population studies.

**1. 为什么要使用非刚性变换？**

- 当您需要考虑**图像中更复杂的物体或区域的形状和结构发生变化**时，就需要使用**非刚性变换**。对于医学图像分析等领域，刚性和仿射变换对于局部变形场景太过受限制。

**2. 非刚性配准的应用（个体内或个体间）：**

- **个体内非刚性配准：** 在这种情况下，您处理的是**同一人或患者在不同时间点的图像**。非刚性配准可用于考虑随时间发生的解剖变化，例如呼吸过程中的器官变形、肌肉收缩或由于疾病进展引起的组织结构逐渐变化。
- **个体间非刚性配准：** 当您有来自不同个体的图像（个体间）时，非刚性配准变得重要，以对齐展示明显解剖差异的图像。例如，对齐具有不同形状和大小的不同大脑的MRI扫描图像，或比较不同患者具有先天性或后天性畸形的图像。

在个体内和个体间的情况下，非刚性配准允许您考虑解剖的复杂和局部变化，从而实现准确的比较、分析和医疗干预。它在处理可变形结构和刚性或仿射变换无法捕捉必要变形的情况下尤为重要。应用范围从疾病监测到手术规划和人群研究。

##### Example of non-rigid registration across several subjects

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/vY3vg4N.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


#### Similarity measures

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/6imIdry.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 7. MSE 
  </div></div>

So let's talk about **similarity measures**.

What could we use as **similarity measure**?

- Absolute difference
- **Sum of squared difference**
- **Cross-correlation**
- Mutual information

There are two types of similarity measures described above? What are the two types?

##### Mean sum of squared differences

$$
MSE = \frac{1}{n}\frac{1}{m}\sum^{n}_{x=1}\sum^{m}_{y=1}\left(I(x, y)-J(x,y)\right)^2
$$

We have one of the most obvious similarity measure is the mean sum of squared differences. So what we do would take the same pixel in each image, and compare the intensity difference and we square it and we sum up all squared differences and we divided by the number of pixels we have.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/vaWcNyY.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 8. MSE calculation 
  </div></div>

So here is an example of these two images and the result of similar measure calculation. And the smaller the value, the most similar the images are. So we want to **minimize** it.

##### Normalized sum of squared differences

The sum of squared differences works well if the images have the same intensity ranges. What if they have **a different intensity ranges**?

One of the good idea which is used almost always when the pixel-wise comparison is used is **normalization**.

What we do is we take our intensities in our image, we compute the mean intensity, we compute the standard deviation. We can test this and we normalise them. From the original images subtract the mean that the new mean will become $0$. And we divide it by the standard deviation.


$$
I^* = \frac{I-\overline{I}}{\sigma(I)}
$$





<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/3w7cKY7.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 9. NMSE
  </div></div>

By doing this, everything is the same except the **intensity range**. And it is more good useful in practical applications.

So **normalization** is one of the things which is very commonly used and this is **pixel-wise similarity measure**.

##### Normalized cross-correlation

Another idea which we can go further in this area, what we can do is we can do the **cross-correlation**.


$$
NCC = \frac{\sum_{x, y}\left(\left(I(x, y) - \overline{I}) \cdot(J(x, y)-\overline{J}\right)\right)}{\sqrt{\sum_{x, y}(I(x, y) - \overline{I})^2\sum_{x, y}(J(x, y)-\overline{J})^2}}
$$



So what we do is to **find the normalized pixel values** and then **multiply each other**, and **find the sum of them**. And then we divide it by the squared root of multiplication of squared difference. So to see how these things behaves, let's go to the mirror board to see some examples.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/p8i5spT.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 10. Cross-correlation
  </div></div>

So, what we would like to do is would like this expression to get good values when we compare the upper two patches, and to have bad values when compared to the other two. We are going to quickly compute them to see what happens.


$$
NCC_{good} = \frac{3\cdot3 + 3\cdot(-1)\cdot(-1)}{\sqrt{\left(9+1+1+1\right)^2}} = \frac{12}{12}=1
$$


What about the bad patches?


$$
NCC_{bad} = \frac{-3+1+1-3}{\sqrt{\left(9+1+1+1\right)^2}} = \frac{-4}{12}=-\frac{1}{3}
$$


What we would like the normal cross correlation to be as high as possible because we saw that for the **same patch**, we get $1$. And for two different patches, we get $-\frac{1}{3}$. And we actually see one of the good property of the normal cross-correlation. It seems to be that its values are actually restricted into the $-1$ and $+1$ range. So when we get to same patches, basically the best possible scenario. We get the result with $1$. So probably if we get the opposite patch, we will probably get $-1$. 

The good thing about is normalise as cross correlation is, that's why the name comes that our values. They **get normalized** into the $-1$ to $+1$ range. This is very **useful** when we compare two patches.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/M4pqEGG.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 11. Maximization
  </div></div>

***Will MSE and NCC work for CT-MR image registration?***

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/SCQfK8H.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 12. CT-MR
  </div></div>

We want to perform **registration** on **CT-MR** image on two images, which are **very different intenstity patterns**. The edges in these images are kind of **similar**. 

You can see that there is a **mandible** on the right chart, which is **black** but on the left side, it is **white**. You can see the edges over human skin. They are also kind of more or less visible. But what is important is that the intensities now is completely different. So the intensitity plan is completely different. 

***Now the question is if you can use MSE or NCC or something like this for this registration.***

Actually turns out we **<u>cannot</u>** use them because what MSE and what all of these metrics for what they tried to do is try to **<u>match high intensity with high intensity, and low intensity with low intensity</u>**.

But in our particular case, what we would like to do is we want to **match somehow the patterns the edges within the pixels**. For example, we want to match a bone, where a black and white intensities in different patches. **It's very difficult to just in advance define this, so we can know what exactly we want to match**.  => *But we want to match patterns not individual intensities.*

##### Mutual image information

For this reason, there is another metric which is called **mutual image information**.

  The metric is most commonly used in **image registration**. It requires a lot of computation resources, but at the same time, it is a very **versatile** can be applied for many different images. I do not need to know many things about images in advance. We can just apply it and it will most likely work.

Here is an equation how mutual information works.


$$
MI = E(I, J)
$$

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/6FN99jJ.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 13. Joint entropy
  </div></div>

So we need to compute the entropy issues and we will compute in histogram manner.

Below is the illustration.

Let's say we have different images which we want to match each other. Let's assume these are the images of some object. 

What we do is we create a 2D plot where at a lower dimension.  

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/oosMzEx.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 14. Illustration
  </div></div>

Then, we will do the same thing for the next pixel, and ..... it will finally come with a cloud of points. And we are interested to how **compact** is this cloud.

From this, if there are two images just inversion of each other, our plots will be very compact. It means, we just need very small regions to cover all the possible pixels. But when the images were random, the pixels which would plot it on this 2D plot, they would distribute to the space in a completely manner. They could be occupied a lot of area.

So what we can do and the idea of the mutual information is to complete a histogram of the pixels in both axises $x$ and $y$ and which is a probability histogram. And then compute probability histogram for joint probability.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/K8wN4pA.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 15. Illustration
  </div></div>

**<u>So the more compact the more higher will be mutual information.</u>**

And here is basically how they look like for this particular image.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/Wuk9Ahc.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 16. Mutual information
  </div></div>

These are with the mutual information for these two images which are just inversion of each other.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/GFL1GWk.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 17. Rotation mutual information
  </div></div>

Here is the rotated version. And you can see that the plot now is no longer compact as it was. It occupies a lot space. And that's why the mutual information is way lower. And our ideas is to **maximize mutual information**.

#### Interpolation

Another concept which we need to know is the concept of **interpolation**.

What different interpolation techniques do you know?

- Nearest neighbour
- Linear interpolation
- Barycentric interpolation
- Spline interpolation

##### Interpolation effects

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/1EVjwqL.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 18. Interpolation effects
  </div></div>

Different interpolation techniques work on diff. images, the choice of interpolation technique can have varying effects or perform differently depending on the characteristics and content of the input images.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/YanUjxl.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 19. Effects of different interpolation techniques
  </div></div>

不同插值技术中提到的 RMSE（均方根误差）值可以作为一种量化指标，用于评估每种插值方法应用于特定图像时的质量或准确性。RMSE 是一种常用指标，用于量化原始图像与插值（转换）图像之间的差异。

The **interpolation** in image registration plays two roles. 

One of them is for **computing the values of pixels**. Let's say if the pixels has a coordinate which do not match original coordinates. For example, one pixel has a coordinate $(0, 0)$. But if we would like to move it a little bit, $(-0.1, 0.3)$. **What kind of value of this pixel have?** Should we just around these numbers to get $(0, 0)$ or should we do something more intellectual. 

所以就是当我们做配准时，我们移动了一点点图像，那么原先点上的像素应该是变成多少呢？使用插值法来计算。

Actually we take account the neighboring pixels if they can play some role in the value of a new pixel. There is one way how we can compute it.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/YYy2YJ4.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 20. Example of interpolation
  </div></div>

We have four points $(0,0), (1,0), (0,1)$ and $(1,1)$. Let's say there are pixels $1$, $2$, $3$ and $4$. And would like to find the value of a pixel which will be located here. And then the question is **what will be the value of the pixel**? We know the intensities of these four pixels, and also the corresponding $(x, y)$ of the target pixel.

So the way to do it is to **calculate the contribution**. The contribution of this pixel to our new pixel. Let's just imagine that this $x, y$ will be very close to $1$.
$$
x, y\approx 0.999
$$
So how similar should be the contribution of the pixel? Each of these four pixels or contribute something to the intensity.

The contribution of $I(0, 0)$:

- Proportional to the opposite "rectangle".


$$
(1-x)\cdot(1-y)\cdot I(0, 0)
$$
The same logical goal for other four pixels. So for pixel $(1,1)$, it's contribution to the intensity of our new pixel through the proportional to this.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/J0lWvji.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 21. Contribution illustration
  </div></div>


$$
\begin{align}
I(\cdot) &= (1-x)\cdot(1-y)\cdot I(0. 0) \\
& + x\cdot (1-y)\cdot I(1, 0) \\
& + (1-x)\cdot y \cdot I(0, 1) \\
& + x \cdot y \cdot I(1, 1)
\end{align}
$$
So the rule is that we compute the **size of these boxes** and then **multiplying them** by the opposite pixel. Then, sum them up and get the final new value of the pixel.

#### Image registration is optimization

We need to try to apply registration algorithm, try to optimise registration process.

So what we can learn until now is we now know two pixels in the space and their neighborhood of them, how to compute how similar are they using different similarity measures based on pixel based comparison, like sum of quare differences or normalized cross correlation, or based on some advanced techniques like mutual information.

We also know how this image moving will change if we move it 7.5 pixels up, how many intensities will change. => Interpolation

So we know all this and we can try to formulate the registration, how the registration should be done.

So what we need to do is we need to **minimize** depending on what kind of similarity measure we use. We want to minimize a distance metric.


$$
\min\sum_{i, j}d\left(I(i, j) - J\left(x(i, j, \theta), y(i, j, \theta\right)\right)
$$

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/fgLvTG6.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 22. Similarity measure
  </div></div>

So here is our formulation of for registration optimization. What we need to find is to find $\theta$. ***The question is how to do it.***

The common way to solve it is using an alogrithm called **<u>gradient descent algorithm</u>**. So let's just choose the most simple similarity measures we have. So, for $d$, let's choose the squared intensity difference. So basically, we just take a pixel and find the difference between each pixel individually, and that's it. No mutual information or anything complex like that.


$$
d(I(i, j), J(x, y)) = (I(i, j)-J(x, y))^2
$$
And $\theta$ will only use one transformation - translation. So basically $F$ from $I$ to $J$ is basically a translation we would translate and the $I$ to some value that the $x$ and $J$ to some value.


$$
f(I, J, \theta) - \text{translation using }\theta \\
f(i, j, \theta) = [i+\theta_x, j+\theta_y]
$$


And this is one of the most simple configuration we can have, and now the question is **how we get optimized solution**? How we can find the optimal $\theta$? And the way to do it is to use the **gradient descent algorithm**.

Then, the formulation of the problem is, **<u>we want to optimize</u>**:


$$
E(\theta) = \sum_{i, j}(I(i, j) - J(x(i, j, \theta), y(i, j, \theta))^2
$$

##### Gradient decent algorithm

By using **gradient decent algorithm**:


$$
\nabla E(\theta) = \frac{\partial E(\theta)}{\partial \theta}
$$
And move our solution  move like take some $\theta$, change our $\theta$ according to the value of the gradient.


$$
\theta_{t+1} = \theta_{t} + \eta\nabla E(\theta)
$$
Usually weighting factor is something very small, like $0.01$. In deep learning, it is called, learning rate. So the question is how to find the gradient of the expression?

In our case, 


$$
\nabla E(\theta) = \nabla \sum_{i, j}(I(i, j) - J(x(i, j, \theta), y(i, j, \theta)))^2 = \\
-\sum_{i, j} 2(I(i, j) - J(x(i, j, \theta), y(i, j, \theta)))\cdot\frac{\partial J}{\partial \theta}
$$
Now, the question is how to differentiate the $J$ against $\theta$. 


$$
\frac{\partial J}{\partial \theta} = \frac{\partial J}{\partial X} \cdot \frac{\partial X}{\partial \theta}
$$
And this is again a complex function, we need to first differentiate it against its coordinates (transformed coordinates), basically aginst $x$ and $y$. 

So if you want to differeniate an image against its coordinates, what it means is actually very simple thing. 

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/Lx6vlHx.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
  </div>
</div>


$$
\frac{\partial J}{\partial x} = \frac{1}{2}(I(x+1, y) - I(x-1, y))
$$


Find two neighboring values for every pixel, subtract them from each other, and then multiply by 0.5. And then we do the same thing for the vertical direction.


$$
\frac{\partial J}{\partial y} = \frac{1}{2}(I(x, y+1) - I(x, y-1))
$$
Below is the illustration.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/YQxD2o5.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 23. Understanding
  </div></div>

This part is easy, but the question is how to find the last part on the abovementioned differentiation, the gradient of our coordinates against our transformation.


$$
T(x, y) = \begin{bmatrix}1 \quad 0 \quad \theta_x\\0 \quad 1 \quad \theta_y\\ 0 \quad 0\quad 1\end{bmatrix}\begin{bmatrix}x\\y\\1\end{bmatrix} = [x+\theta_x, y+\theta_y]\\
\frac{\partial X}{\partial \theta} = \begin{bmatrix} \frac{\partial X}{\partial \theta_x}; \frac{\partial X}{\partial \theta_y}\end{bmatrix} = [1, 1]
$$
 Now, we have everything.


$$
\frac{\partial J}{\partial \theta} = [\frac{1}{2}\left(I(x+1, y\right) - I(x-1, y), \frac{1}{2}(I(x, y+1) - I(x, y-1))]
$$
This is just differentiaion of the image because our transformation is just translation. Then, we can compute the whole graient of our system.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/RkzH4GA.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 24. Whole gradient
  </div></div>

Iteration and computation until it stops.

##### Problem with optimization

This is a very expensive procedure to do as these similiarty measures $d(I, J)$ can be complex and expensive to compute (mutual information - regions around pixels).

Another issue is that $\theta$ can contain many variables. In reality, $\theta$ can have thousands of variables, so each individual pixel can move individually. So each pixel will have transformations. Unlike in our case, only two variables.

- 9 for similarity transformations

  
  $$
  \begin{bmatrix}
  \theta_1 \quad \theta _2 \quad \theta_3\\
  \theta_4 \quad \theta_5 \quad \theta_6 \\
  \theta_7 \quad \theta_8 \quad \theta_9
  \end{bmatrix}
  $$

- thousands for non-rigid transformations

**How to simplify this?**

What people usually do is they use so-called **multi-resolution pyramid registration**.

##### Multi-resolution pyramid registration

- Downscale (and smooth) reference and moving images x16 (types)
- Register them and memorize transformation
- Downscale (and smooth) reference and moving images x8, apply memorized transformation
- Register images and combine transformations from both steps
- Continue

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/PiCMkeP.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 25. Multi-resolution pyramid registration
  </div></div>

"多分辨率金字塔配准" 是一种在图像配准中使用的技术，特别是用于将两个具有不同分辨率或尺度的图像对准。它采用分层方法逐步对齐和优化两幅图像之间的配准。让我解释一下您提供的要点，以说明这个技术的工作原理：

1. **将参考图像和移动图像降采样（并平滑）16倍（类型）**：
   - 首先，参考图像和移动图像都会降低分辨率，降低16倍，并可选择平滑以减少噪音。
   - 这一步创建了图像的低分辨率版本。
2. **将它们进行配准并记忆变换**：
   - 对降采样的参考图像和移动图像进行配准，即调整它们的空间对齐以找到最佳匹配。
   - 记忆或保存调整这些低分辨率图像的变换。
3. **将参考图像和移动图像降采样（并平滑）8倍，应用记忆的变换**：
   - 参考图像和移动图像再次降低分辨率，这次降低8倍。
   - 先前记忆的变换（来自步骤2）被应用于这些低分辨率图像。
   - 这一步创建了更高分辨率但仍然减小比例的对齐图像。
4. **对图像进行配准并结合来自两个步骤的变换**：
   - 来自步骤3的更高分辨率参考图像和移动图像进一步配准，进一步精细调整它们的对齐。
   - 从这个配准步骤获得的变换与步骤2中的记忆的变换组合在一起。
   - 这些变换的组合确保图像在更高分辨率下准确对齐。
5. **继续**：
   - 这个过程以迭代方式继续，图像逐渐降低分辨率，逐步对齐和变换。
   - 在每个级别上，变换得到进一步的精细调整和组合。
   - 这一过程持续进行，直到达到最高所需分辨率或达到收敛条件。

**关键点**：

- 多分辨率金字塔配准是一种处理具有不同细节水平和分辨率的图像的常见策略。
- 它从低分辨率开始进行粗略对齐，然后随着考虑到更高分辨率逐步优化对齐。
- 这种方法可以显着加快配准过程，提高对齐的准确性，特别是在处理大型或复杂图像时。

总的来说，多分辨率金字塔配准是一种强大的技术，用于稳健地对齐具有不同尺度或细节级别的图像，尤其在医学影像、计算机视觉和遥感等领域，其中图像可能具有不同的分辨率或包含细节信息。

When you even do pyramid approach, there is still a problem in computation.

In the pyramid approach, people don't do it for every pixel, people do it for a grid. So you just define a grid of points. For example, the size of the image is $100 \times 100$, but the number of grid points is $10 \times 10$. So we just place great place grid points at $10 \times 10$, skip every pixels and base with grids. And what we do is we compute our similarity measure only for this points. And we compute the optimization only for these points. And then we somehow inerpolate the transformation for the internal points.

<div align=center><div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/DV1yHKh.png" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 26. Grid-based registration
  </div></div>

- Do not perform registration per pixel but use a sparse grid

