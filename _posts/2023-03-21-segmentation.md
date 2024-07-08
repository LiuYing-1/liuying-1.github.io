---
layout: distill
title: Segmentation
description: Image Segmentation, including Hough Transform
tags: sip
categories: study ucph
giscus_comments: true
related_posts: true
date: 2023-03-21
thumbnail: assets/img/segmentation.jpg

authors:
  - name: Ying Liu
    url: "https://di.ku.dk/Ansatte/forskere/?pure=da/persons/762476"
    affiliations:
      name: DIKU, UCPH

toc:
  sidebar: left
---

<div align=center><img src="https://i.imgur.com/z0885iZ.jpeg" alt="1551680875610_.pic" style="zoom:80%;" /></div>

<div class="reference" style="color: #999; font-size: 0.8em; margin-top: 1em; margin-bottom: 1em; ">Reference: Lecture from <a href="https://di.ku.dk/english/staff/?pure=en/persons/254908" _target="blank">Stefan Horst Sommer</a></div>

## Segmentation Overview

<div align=center><img src="https://i.imgur.com/IUmbCe9.png" alt="image-20230321111305719" style="zoom:33%;" /></div>

In this example, you have an image that is filled with a lot of triangles. And basically, we wanna **<u>extract those coherent structures from the image</u>** and there are multiple ways of doing that.

Another example is an image with a collection of coins. Here you can actually see that you have a uniform background where the foreground actually varies in intensity.

<div align=center><img src="https://i.imgur.com/cekguer.png" alt="image-20230321111514252" style="zoom:25%;" /></div>

More complicated patterns are textures. Here, for instance, this is a collection of trees trunk.

<div align=center><img src="https://i.imgur.com/OtCGHmA.png" alt="image-20230321111704852" style="zoom: 25%;" /></div>

And in many cases, some of these scenes are quite complicated. So here is an example over here. You can see that you have a complicated scene of a horse back rider with a horse. There's a flag here. There is a selection of cars up here. There's a shrubbery and so on.

<div align=center><img src="https://i.imgur.com/rFKCqS9.png" alt="image-20230321111900741" style="zoom: 50%;" /></div>

So there's of course different interpretations of this.

The ground truth here annotated you can see that you regard the shorse as a separate object and the rider is separate object. The cars are separate objects but the same type. And you can also see this a spectator or something marked here. (Just a person here)

The results from two different types of segmentation FCN-8s and the other one.

From FCN-8s, you can see that you very clearly get the horse and the rider. It doesn't have the same fidelity as a ground truth, but it's pretty close.

While SDS here, you can see that you get some of the horse, you get some of the rider, and you also get the cars, but it's not clear. So => It is not easy.

---

The next example here you can see two motorbikes.

And again you have the bikes same class. And then, you have the humans. Spectators, the riders and the spectators. You can see here that the two methods, the convolution neural network here does a really good job. But it defines the bikes and the riders whereas spectators are more shady.

In SDS case, you can see that the riders and the bikes are melded together and you find some structure in the background, but that is not really the spectators.

---

The next one here you can see another difficult example you have a sheep. And a wire fence.

Whether this method is capable of reasonably delineating the sheep but completely misses out on the wireframe. And the other method, here is a capable of capturing most of the sheep but due to the wire for the wire frame, it's simply cutoffs some stuff here.

---

The final one is a boat.

In the FCD you can see that you have three classes and curious where they come from but at least given the ground truth.

Here you can see that the top of the boat which is basically water here. => SDS not work.

---

Other examples include segmentation of anatomical structures. So this is really common in medical image analysis. So in this case, we are interested in two specific regions of the brain.

One is Ventricle and the other one is the left cuneus and what you can see here is that it's not necessarily an easy task depending on what kind of method that you use.

<div align=center><img src="https://i.imgur.com/p6K72fU.png" alt="image-20230321114210885" style="zoom: 33%;" /></div>

A more complicated scenarios when you have multiclasses in this case we know there's 132 classes.

<div align=center><img src="https://i.imgur.com/XD2y0pv.png" alt="image-20230321114656540" style="zoom: 33%;" /></div>

This is a more complicated nature.

Another example which is also well described in the literature is the segmentation of Hippocampus for AD diagnosis/prognosis.

<div align=center><img src="https://i.imgur.com/JrWU27S.png" alt="image-20230321114906910" style="zoom: 33%;" /></div>

So the hippocampus is the structure that is related to memory. When you get Alzheimer's, what will happen is that you can observe that people basically use the ability to remember things and it's strongly correlated with a reduction in the hippocampus.

Another example here is knee segmentation and here we are interested in the cartilage. Actually this is the particular intersection between the two cartilage sheets and either side of the knee.

<div align=center><img src="https://i.imgur.com/VJzDmXj.png" alt="image-20230321115242548" style="zoom: 33%;" /></div>

And here you can see a complete reconstruction of the two different sheets of cartilage.

Another example is segmentation of airways.

<div align=center><img src="https://i.imgur.com/4fejT0w.png" alt="image-20230321115530862" style="zoom:30%;" /></div>

So, from a CT image, basically extracting the airways and classify them into branches. This is the evaluation of people belong to disease. This is probably very hot with respect to the current Corona virus.

---

So the learning goal is defining the segmentation problem, so actually sort of confining, what is it really that you want to do?

- Constrast local/edge/region-based and semantic segmentation
- Select appropriate segmentation methods
- implement simple segmentation algorithms
- define and implement line Hough transform
- formulate the segmentation problem in a machine learning setting
- use machine learning based segmentation
- solve a specific real-life segmentation problem

---

### Segmentation Approaches

So a really good example is .....

If you think of edges of zebras here, the segmentation was used to animal is a inherentely diffcult problem. Because you can see we have a lot of it, just not only the outline of the animal itself, but due to the texture of the animals. Region based methods or methods of general are easily fool and you can even see that's a background here has been intuitive in the segmentation.

<div align=center><img src="https://i.imgur.com/zA2zF86.png" alt="image-20230321120729252" style="zoom: 25%;" /></div>

#### Intensity based Segmentation

So, if we turn ourselves to a grayscale. If we just assume a simple thresholding.

<div align=center><img src="https://i.imgur.com/1woCqA6.png" alt="image-20230321121144138" style="zoom: 25%;" /></div>

You can see here that if there is a bias through the image, we have a problem right? Because then some of the foreground pixels as you can see here were falling into the intensity space of the background pixels.

So what will happen is that we will have basically get a misclassfication over here.

A way to get rid of this is simply using some of the methodology that you already know.

I will rectify the image, that is basically model this distortion like you see, so that the intensity of the different levels are leveled. In this case, you are interested in the background and foreground.

And you can use that to sort of estimate a function that will intensity correct the image so that you have a uniform intensity distribution.

If you do that, you will end up with examples more like this.

<div align=center><img src="https://i.imgur.com/shaOKyJ.png" alt="image-20230321121910947" style="zoom: 33%;" /></div>

But what you can see from this is that, if this is your image, and you blur it. You can see here that you have two distinct peaks which is the background here where the intensities are zero. And you can see here you have the foreground which are very distinct and then you need to choose some sort of threshold in between here.

When you select this, it has an effect. So in this case, due to the blurring, you get rounded corners. The objects become smaller and smaller if you go to the right part to choose $\tau$, while if you go to the left part, the objects become larger and larger but it will not sort of reflect the real size.

So this is one of the basic problems with intensity based and this is of course based on the resolution of different images.

So what you really can consider this as is this will be the original image and this is sort of a low resolution because if that's been smoothed of this one. And when you do the segmentation, you get artifacts so these are you can see that these convex parts of the shape sort of gets blurred away and concave parts is sort of diminished right. So things tend to be rounded.

Another way to do this is to use K-means clustering.

<div align=center><img src="https://i.imgur.com/t7IzaY3.png" alt="image-20230321124355857" style="zoom: 33%;" /></div>

So assume that you have corrected your image so that you don't have any bias. You can run K-means clustering on the intensity space.

In this case you can see that if we choose 9 clusters, we can see some artifacts of the background out here. And there's some speckle noise. Well, this brain is divided into this background and in two classes and then you would have seven different classes in here.

And here you can see one of the clusters and another output of clusters.

#### Edge-based Segmentation

<div align=center><img src="https://i.imgur.com/m15SYk6.png" alt="image-20230321125146780" style="zoom:50%;" /></div>

So if we take this original image what we have here is a binary image, and there's a very nicely geives these two peaks. But if you have like we saw in the green image, sort of that biased throught the image, you can see that stalled distribution very clearly.

However, there are methods that relies on edge detection that can be used to mitigate this.

So, by running an edge filtering, you can see that we can actually get almost the same kind of accuracy in the segmentations through here and this is because the transition become from background to foreground. Can be described by the derivatives of the image and exactly at the edge of this is where the derivatives has the largest magnitude. And if you think of the second order derivative, then that's the change of the derivative, the ridge of that would be exactly on the edge. So what you are basically get is a sort of a bump right here.

And you can use this to extract the edges.

So, to extract the contours of the object that relent to edge tracking and what is described int the text you need to read is you will basically search for the highest magnitude of the gradient or this edge detector and then you will basically trace it around. Just searching for the largest and you stop once and you can't find the neighbor.

<div align=center><img src="https://i.imgur.com/V9c3dHK.png" alt="image-20230321132310162" style="zoom:33%;" /></div>

So if you want to do the whole outline you have to do like edge tracking, another example of doing this is the watershed, which also will basically track the edges of the object.

<div align=center><img src="https://i.imgur.com/d1gVI4I.png" alt="image-20230321132638404" style="zoom:25%;" /></div>

Pyramid linking is also a way to sort of create coherence across scales and the way to do this is simply to ....

So if you imagine that you first create the gaussian pyramid and the way it's described in the book is that you have sort of an intrinsic linking right.

So at the highest level here, it is result of the blurring of the next level.

<div align=center><img src="https://i.imgur.com/LjJL9Yd.png" alt="image-20230321133416869" style="zoom: 25%;" /></div>

<font face="noto serif sc">金字塔链接（pyramid linking）是一种在图像和信号处理中常用的技术，用于构建图像金字塔或信号金字塔。金字塔是一种多尺度表示，用于在不同的尺度上分析信号或图像。通过金字塔链接技术，可以将不同尺度的金字塔层级相互连接起来，以提高图像或信号的分析能力。它可以将不同分辨率的图像或信号组合在一起，从而创建一个具有不同分辨率的层次结构。在这个层次结构中，每个级别的图像或信号都是前一个级别的下采样版本。</font>

<font face="noto serif sc">在图像处理中，金字塔链接通常是通过将图像进行降采样来实现的，即通过将图像缩小到不同的尺度来构建金字塔层级。每个金字塔层级都包含一个缩小版本的原始图像，而缩小的尺度越高，图像的分辨率就越低。金字塔层级可以通过卷积、滤波、差分等方法来计算，以提取不同尺度的图像特征。Pyramid linking通常用于图像处理中的特征提取和匹配，以及信号处理中的滤波和分析。它可以帮助识别图像中的不同特征，并在不同分辨率的图像或信号中找到相应的特征。这使得它在许多计算机视觉和图像处理应用程序中非常有用，例如目标检测、图像匹配和纹理分析。</font>

#### Region based segmentation

Another method is the **Chan-Vese** which is a segmentation based on active contours, and what you are basically have is a function that continuously moves through the instruments so we start out here by initially a circle.

This is actually in fact an implicit function so it's positive inside and negative outside. That gives us a binary segmentation and what it minimizes is a variant in the outside and the variance in the inside. And because it's an implicit function, it gets sort of crawl across boundaries as you can see here and feel this. So the solution here found by the Chan-Vese, the one that minimizes the variance outside plus the variance inside.

<div align=center><img src="https://i.imgur.com/GSTfAau.png" alt="image-20230321134734510" style="zoom: 33%;" /></div>

**_Active contour is the predecessor of Chan-Vese method._**

<div align=center><img src="https://i.imgur.com/35iV3Vf.png" alt="image-20230321134840903" style="zoom:50%;" /></div>

The idea was to have a line or contour thta could cling to edges in order to be able to model both concave and convex of the objects and separate them.

There is an example that the contours clinging to the edge here and you can then extract it and use a prior that requires a certain smoothness which will make it collapsed back onto the edge of the object.

But we usually use Chan-Vese as it already does a very excellent job.

#### Registration based

<div align=center><img src="https://i.imgur.com/iPDuhtQ.png" alt="image-20230321135312444" style="zoom: 33%;" /></div>

We can think of this as an example method in terms of machine learning. So what you have is you have a collection of atlases.

So basically images that have already been segmented at all parts of the structure. Then you perform the registration between the target image with the structure that you're interested in and the atlases. And then you do a label propagation so that means once these things are placed on top of each other sufficiently. Optimal you propagate the labels from the addresses to the target image and then you can use multiple techniques to combine the labels from the atlases to the final segmentation.

## Hough Transform

### Line

<div align=center><img src="https://i.imgur.com/qBJu2XD.png" alt="image-20230321142325992" style="zoom: 33%;" /></div>

So the Hough Transform is a mapping of lines from the first domain to the second domain.

It's basically you draw lines in this space, measure the intensity (plot in the second image), that's the parameterized by the distance of the line to the center of this image plus the slope which you can see here.

And from that, when you do that a lot of times, you get poles in the second image which corresponds to an angle.

<font face="noto serif sc">霍夫变换（Hough Transform）是一种图像处理技术，用于检测形状在图像中的出现，并将其表示为数学形式。它最初是由霍夫在1962年提出的用于检测直线形状的算法，但后来被拓展用于检测其他形状，如圆形和椭圆形等。</font>

<font face="noto serif sc">在第一个图中，白线组成的方框图包含了若干条直线。霍夫变换的主要思想是将这些直线从原始图像空间（x,y）映射到另一个参数空间（θ，ρ），其中θ是直线的角度，ρ是直线到原点的距离。因此，第二个图展示了这些直线在参数空间中的分布情况，每个点代表一条直线。</font>

<font face="noto serif sc">在第二个图中，可以看到一些明显的聚集点（也称为“极值”），这些聚集点对应于原始图像中的直线。这些聚集点可以通过设定适当的阈值来筛选出来，从而得到第三个图中的检测到的直线。</font>

<font face="noto serif sc">总之，霍夫变换是一种用于检测形状在图像中的出现的技术，它通过将形状从图像空间映射到参数空间来实现。在参数空间中，形状出现的位置可以用极值来表示，这些极值可以用于检测形状并提取相关信息。</font>

<div align=center><img src="https://i.imgur.com/OqrWeSA.png" alt="image-20230321144427216" style="zoom: 22%;" /></div>

Have an image space looks like this, and through all the points you can draw multiple lines. So, for each line, you get a slope $m$ and a bias $b$. That gives you a unique mapping to the parameter space.

What is really interesting in the second image is that the places where these lines intersect is exactly the coefficient of the line that passes through these two points.

So, in practice, what is done is similar to what is up (previous example) here where this paramemterized in terms of the distance to the center and the slope in degrees.

**Steps to do Hough Transformation**

**_Step 1_**: Edge detection

**_Step 2_**: Then you need to decide on the number of angles $(ntheta)$ and distances $nd$. So basically parameterize your Hough space. Create accumulator image $H[ntheta, nd]$.

**_Step 3_**: And then you simply start voting for each edge and theta calculate $d$ and increment $(theta, d)$ pixel in accumulator.

So at the end, **_you find peaks in accumulator image._**

### Circle

**Extension on circle**

<div align=center><img src="https://i.imgur.com/mKOcI4N.png" alt="image-20230321150955054" style="zoom:30%;" /></div>

So, here you extract boundaries of the coins using an edge filter. So you get these here. For instance, you can parameterise this by the distance to the center and the size of the circle.

### Segmentation Evaluation

**_Once you have a segmentation, how do we actually evaluate it in this case?_**

**_Dice Measure_**

$$
Dice (A, B) = 2 * |A\cap B|/(|A| + |B|)
$$

The overlap between $A$ and $B$. How well do they overlap?

## Machine Learning based segmentation

So, in a supervised setting, you have $X$ features and $Y$ output, like a regular regression model. And the learning algorithm searches for a mapping from $X\to Y$ in the hypothesis set $H$.

The segmentation we would have the full image would be the $X$ image, $Y$ is the segmented image (labelled image).

If it's a patch-based segmentation, it would be a patch of an image and $Y$ could be the class of center of that pixel.

### Supervised Learning

<div align=center><img src="https://i.imgur.com/dSVUZYM.png" alt="image-20230321152153638" style="zoom: 25%;" /></div>

The idea is that you don't know your function that takes you from $X \to Y$, but what you do have is you have a set of examples which is called training data. And from that, you try to select from your hypothesis that which is basically there are the set of candidate functions. You try to select the optimal to the final hypothesis.

### Convolution Neural Network

<div align=center><img src="https://i.imgur.com/3gAxVpr.png" alt="image-20230321152507729" style="zoom:30%;" /></div>

### Fully Convolutional Networks

<div align=center><img src="https://i.imgur.com/QTZTtLI.png" alt="image-20230321152645148" style="zoom:33%;" /></div>
