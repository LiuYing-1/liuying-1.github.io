---
layout: distill
title: Filtering
description: Linear and non-linear filtering, derivative operators.
tags: sip
categories: study ucph
giscus_comments: true
related_posts: true
date: 2023-02-15
thumbnail: assets/img/filtering.jpg

authors:
  - name: Ying Liu
    url: "https://di.ku.dk/Ansatte/forskere/?pure=da/persons/762476"
    affiliations:
      name: DIKU, UCPH

toc:
  - name: Definitions
  - name: Applications of filtering
  - name: Linear filtering
    subsections:
      - name: Linear continuous filter
      - name: Discrete filtering - Intuition
      - name: Filtering with a sliding neighborhood window
      - name: Linear discrete filtering
      - name: Discrete filtering for one pixel in output image
      - name: Some common linear filters
      - name: Discrete Mean filter
      - name: Filtering at the border of the image
      - name: Linear separable filters
      - name: Discrete Gaussian filter
      - name: Noise removal by linear filtering
      - name: Correlation
  - name: Non-linear filtering
    subsections:
      - name: Definitions
      - name: Example of a non-linear filter - Median / rank filtering
      - name: Noise removal by non-linear filtering
      - name: Median filtering of different salt and pepper noise levels and filter sizes
  - name: Derivative filters
    # subsections:
    #   - name: Image as arrays or functions
    #   - name: Approximating image derivatives
    #   - name: Other first order derivative filters
    #   - name: Linear seperable filters
    #   - name: Computing derivatives with Sobel filter
    #   - name: Differentiation filters amplifies noise
    #   - name: A snippet of differential geometry
    #   - name: Edge detection
    #   - name: Zero crossings of Laplacian of Gaussian filter
  - name: Filtering color images
---

<div align=center><img src="https://i.imgur.com/v5mpAt9.png" style="zoom: 80%"></div>
<div class="reference" style="color: #999; font-size: 0.8em; margin-top: 1em; margin-bottom: 1em; ">Reference: Lecture from <a href="https://di.ku.dk/english/staff/vip/researchers_image/?pure=en/persons/83684" _target="blank">Kim Steenstrup Pedersen</a></div>

## Filtering

Convolution is no longer a pixel-wise operation but I operate functions of images that require several pixels in order to compute a value for one pixel.

### Definitions

**Filtering** is a fundamental part of signal and image processing.

It (a **filter**) takes us its input as a signal or image if we like and produce as the output another processed signal or image.

It's a function of an image and its output is an image.

Filters on images are defined on **image neighborhoods**.

There are different types of filtering, **linear filtering**, and **non-linear filtering**.

The most common filters are **linear and shift invariant** (LSI) - i.e. the filter does not depend on position and the filter is a linear operator and is defined by convolution integral and the $h$ (PSF) function.

### Applications of filtering

- **Noise reduction**

  Noise often has high frequency, and it can be reduced by **a low-pass filter** effectively removing high frequencies.

- **Contrast enhancement**

  To improve bright and dark areas in image make it more visible.

- **Sharpening / deblurring / deconvolution**

  If you have a blurred image, you can perform sharpening / debarring / deconvolution. The end goal is to construct a more **sharp version** of the original image.

- **Detection of edges and other features**

  For analysis purposes, for instance, detect something called features which could be intensity **edges** and so on.

- **Recognition by templates**

  Recognize **certain shapes** of parts of an image and one way to do this is using **templates**.

- **Prediction / Recognition / Detection of materials / Objects in images**

### Linear filtering

**Recap - Linear shift-invariant systems => the convolution integral**

A **linear system** is defined by a **linear operator** $S$.

An operator $S$ is **linear**, if $S\{aX+bY\} = aS\{X\}+bS\{Y\}$.

The linear operator $S$ is defined by the linear superposition integral.

We say that the linear system and PSF is **shift-invariant** if

$$
h(x, y;x',y') = h(x-x', y-y')
$$

This leads to the **convolution integral**.

$$
g(x, y) = \int^{\infty}_{-\infty}\int^{\infty}_{-\infty}f(x', y')h(x-x', y-y')dx'dy'
$$

#### Linear continuous filter

Consider an image defined by the function $f(x, y):\Omega \mapsto \R$ where $\Omega \subseteq \R^2$. For simplicity, we consider a 1 channel image, e.g., a real-valued grayscale image.

<div style="font-family: 'Noto Serif SC'">两个坐标指向一个像素点的位置中包含的灰度值（简单起见）。</div>

A linear filter is defined by an **impulse response** or **PSF** given by the function $h(x, y): {H}\mapsto \R$ where $H \subseteq \R^2$.

Applying the linear filter to our image $f$ is done by **convolution**

$$
g(x, y) = \{f*h \}(x, y) = \int\int^{\infty}_{-\infty}f(x', y')h(x-x', y-y')dx'dy'
$$

The resulting function $g$ is the filter's response to the image $f$.

The support of the image, $\Omega$, and the filter, $H$, does not have to be equal.

**Note:** We also need to assume that both functions are square-integrable, $\int\int_{-\infty}^{\infty}|f(x, y)|^2dxdy < \infty$.

#### Discrete filtering - Intuition

In discrete filtering, we compute a new value for a **target** pixel as a function of the neighboring pixels.

An **image neighborhood** is defined by a local window of $N\times M$ pixels that are all connected neighbors.

Common to use **square** windows, $N\times N$.

An example of a $3\times 3$ pixel neighborhood.

<div align=center><img src="https://i.imgur.com/yzup0Mu.png" /></div>

In the **neighborhood window** we also need to define a filter **center** which **coincide with the target pixel**.

If the window size is odd the center is usually at the center of the filter window. But the center could be anywhere in the window.

For instance, if we use NW pixel here, we could consider not the actual neighbors around the target, but the pixels that are to the right and below the target because then, the target would be right here.

#### Filtering with a sliding neighborhood window

How we apply the filter to an image is that, we position our neighborhood window somewhere. We assume that our target pixel is where I put the red dot. And that **the center of the mask is also in the middle of the window**. => So these two things overlap and it means that we are considering all the pixels in this window that we do some function of these pixel values to compute a value for the pixel in the middle.

So we are creating a new image where in the new image, the pixel at that location would be a function of all the pixels here in the input. And it's normal to have this sliding window apprach where you move this window across the image.

It's natural to take stride of 1 pixel so that you move the window by shifting one pixel.

But in principle, we could take larger strides. In this case, we'are ending up with the resulting image that comes out of the filter has a lower resolution than the input.

If the stride is 2, then, we are halving the width and the height of the image.

<div align=center><img src="https://i.imgur.com/CLpWR2Y.png" alt="image-20230214195900647" style="zoom:67%;" /></div>

#### Linear discrete filtering

Consider a discrete image defined by the function $f(x, y):\Omega \mapsto \R$ where $\Omega \subseteq \Z^2$. For simplicity we consider a real-valued 1 channel image. $\Z$ represents the integers.

A linear filter is defined by an **impulse response** given by the function $h(x, y): H\mapsto \R$ where $H \subseteq \Z^2$.

If $h$ has finite support (finite impluse response (RIR)) with neighborhood window size $N\times M$, then applying the linear filter to our image $f$ is done by discrete convolution

$$
g(x, y) = \{ f*h\}(x, y) = \sum^{\lfloor N/2\rfloor}_{i=-\lfloor N/2\rfloor}\sum^{\lfloor M/2\rfloor}_{j=-\lfloor M/2\rfloor}f(x-i, y-i)h(i, j)
$$

Notice the convention of flipping $f$ instead of $h$. => It's equivalent to either do the shifting in each. They lead to the same result. So when programming, we need to consider whether it's easier to flip the image or the filter kernel.

#### Discrete filtering for one pixel in output image

<div align=center><img src="https://i.imgur.com/7yxuDUZ.png" alt="image-20230214203115471" style="zoom:67%;" /></div>

We could think of it as we take the block of pixels around the target and then we flip it left to right and upside down as well. Then, we start to do the product.

So, image in, we need to define an impulse function $h$ - FIR and represent it with some filter mask / kernel and by applying convolution, we get a new image out => $g$.

<div align=center><img src="https://i.imgur.com/yMaEPaV.png" alt="image-20230214203919061" style="zoom:50%;" /></div>

So as before what happens when you do convolution, you consider all the pixels in the image. You consider all the pixels in the image so you slide your filter kernel across the imaeg and that every location you do this weighted sum that gives you a number that you can write into the output image.

#### Some common linear filters

**Example of last week** - Convolution with a continuous shifted box filter $h(x)$.

<div align=center><img src="https://i.imgur.com/vxuNVSj.png" alt="image-20230214204407238" style="zoom:67%;" /></div>

We have two things, a signal or an image $f$, and an impulse response function $h$ which is called the **box filter**.

**_How this box function could be defined?_**

So, in 1D, the box function could be defined as,

$$
h_w(x) = \begin{cases}
1, |x| \leq w/2 \\
0, otherwise
\end{cases}
$$

$|x|$ represents the absolute value of $x$. $w$ is the width of the box. Filter center at $x = 0$. And this says the center of this filter is given at $x$ equals $0$ which is in the middle of the box function.

<div style="font-family: 'Noto Serif SC'">Box filter是一种用于图像处理和计算机视觉中的平滑滤波器。它是一种线性滤波器，使用具有相同权重的正方形内核来平滑图像中的像素。Box filter的内核通常是一个归一化的矩形或正方形，其中每个元素的权重相等，因此也被称为均值滤波器。Box filter通常用于降噪或模糊图像。在计算机视觉中，它也可以用于图像缩放、边缘检测和其他一些算法中。Box filter的缺点是它的平滑效果较弱，不够柔和，在对于较小的图像细节和纹理等细节的处理上表现不佳。在实际应用中，通常会结合其他滤波器来获得更好的平滑效果。</div>

#### Discrete Mean filter

Away from $x = 0$, we can also form a discrete box filter => **Discrete Mean Filter**.

The mean filter is basically a normalized discrete finite support version of the continuous box filter function.

A discrete mean filter kernel ($N \times M$ pixel neighborhood window) is a constant kernel with weights $h_{ij} = 1/NM$.

Computes the mean of the intensities in the neighborhood window.

Example: A $3\times 3$ mean filter kernel.

<div align=center><img src="https://i.imgur.com/bS4kkve.png" alt="image-20230214211637411" style="zoom:67%;" /></div>

<div style="font-family:'Noto Serif SC'">Discrete mean filter，也称离散均值滤波器，是一种常见的图像平滑滤波器。它使用一个正方形的窗口（通常称为内核或卷积核）滑动遍历整个图像，在每个位置计算窗口内像素的平均值，然后将该平均值作为该位置的像素值。具体地，离散均值滤波器的内核大小是一个具有相同权重的矩形或正方形，该内核的每个元素的权重相等，通常被称为"盒子"或"箱子"（box）内核，因此也被称为"盒子滤波器"（box filter）或"均值滤波器"（mean filter）。离散均值滤波器的优点是简单易懂、易于实现、计算速度快，适用于各种类型的图像和图像噪声。但是，由于使用均值滤波器可能会导致图像模糊、损失图像细节和边缘信息，因此在一些应用场景下需要使用更高级的滤波器。</div>

When you do convolution with this filter, you are taking the average of the intensities of the image underneath this filter.

You can think of the $n$ in this discrete version of the box filter as being equivalent to the parameter $w$ in the continuous box filter.

#### Filtering at the border of the image

At the image border, parts of the filter kernel will be outside the image - i.e. we have no pixels to compute the weighted sum of.

Possible solutions:

- **Pad the border** with a constant value, usually zeros (means that the contribution from outside the image is zero).
- **Symmetric mirroring** of image pixels - extend the image by padding with pixels by mirroring the pixels at the image border.
- **Periodic boundary** - the image is wrapped onto a sphere. Filtering on the right border requires pixels from the left border.
- Leave **border pixels unchanged** (copy from input to output)
- Filter only inside the image and **crop** away the border (the output will be smaller than input image)

What you should choose depends on you application.

#### Linear separable filters

The box filter, the mean filter is a kind of special filter. It's linear separable.

A linear filter is **linear separable**, if it can be **decomposed into a linear filter along the x-axis and another along the y-axis**.

Not all linear filters are separable.

This can make filtering implementations more efficient - two $1D$ filtering operations ($2N$ Arithmetic operations) instead of a $2D$ filtering ($N^2$ arithmetic operations)

The $2D$ box filter is separable and can be expressed by the convolution of two $1D$ box filters

<div align=center><img src="https://i.imgur.com/QXOvjxZ.png" alt="image-20230214215124119" style="zoom:67%;" /></div>

**_Proof_**

$$
(\delta_{(0, 0)}*h_N) * h_N^T = h_N * h_N^T = h_{N\times N}
$$

#### Discrete Gaussian filter

The discrete Gaussian filter is a discretized finite support (FIR) version of the continuous function

<div align=center><img src="https://i.imgur.com/PlLgeWd.png" alt="image-20230214215948189" style="zoom:67%;" /></div>

The standard deviation $\sigma$ is the scale/width of the filter. $\sigma$ increased, wider filter.

The Gaussian filter is linear separable and we have

$$
G_\sigma (x, y) = G_\sigma(x)* G_{\sigma}(y)
$$

Example: For $N = 3$ and $x = [-1, 0, 1]$ we have

$$
G_{\sigma}(x) = [0.27, 0.45, 0.27]
$$

Every time ($x = -1, 0, 1$), I get a new value that ends up being this vector or list of values that represents my finite filter.

Usually $N$ is chosen as a function of $\sigma$, e.g., $N=k\sigma$, where $k = 1, 2, or \ 3$.

**Notice the values have been scaled to fit in uint8.**

Examples of 2D Gaussian filter kernels

<div align=center><img src="https://i.imgur.com/05AT6WM.png" alt="image-20230214221441449" style="zoom: 67%;" /></div>

This filter is similar to the Mean Filter, the only difference is that we actually now weigh pixels that are closer to the center more than pixels that are further away from the center of the filter mask.

<div align=center><img src="https://i.imgur.com/m8HzuJw.png" alt="image-20230214221830753" style="zoom:50%;" /></div>

Increasing $\sigma$ => wider the kernel

<div style="font-family: 'Noto Serif SC'">模糊粒度更细</div>

#### Noise removal by linear filtering

**Different noise sources.**

<div align=center><img src="https://i.imgur.com/koTmbPh.png" alt="image-20230214224552142" style="zoom:67%;" /></div>

**Salt and pepper**

The mean filter is just averaging out values. That means, all white or all black pixel will be draged towards the average of a value in the neighborhood around the pixel so that would mean that it becomes more grayish.

For Gaussian filter, we are doing the weighted average and this gives out a slightly more blurred version of the noise compared to what the mean filter does.

**Gaussian** shows the same.

We started out with some noise versions of the original, we applied a filter and it comes with the cost that there's some smearing / blurring of the structures in the image. It has some degree of noise reduction but not complete noise removal.

#### Correlation

The convolution integral

$$
g(x, y) = \{f * h \}(x, y) = \int^{\infty}_{-\infty}\int^{\infty}_{-\infty} f(x', y')h(x-x', y-y')dx'dy'
$$

The correlation integral

$$
g(x, y) = \{f \circ h \}(x, y) = \int^{\infty}_{-\infty}\int^{\infty}_{-\infty}f(x', y')h(x+x', y+y')dx'dy'
$$

Difference is whether we flip $h$ or not as well as some theoretical consequences.

Correlation is often used to implement discrete convolution, and instead flip filter kernel $h$ or image $f$ before correlation.

$$
g(x, y) = \sum^{\lfloor N/2 \rfloor}_{i=-{\lfloor N/2 \rfloor}} \sum^{\lfloor M/2 \rfloor}_{j=-{\lfloor M/2 \rfloor}}f(x+i, y+j)h(i,j)
$$

<font face="Noto Serif SC">我的理解是，卷积是翻卷再 correlation。</font>

If you have a fast implementation of correlation, you can always implement a discrete convolution simply by using correlation. The only thing you have to do is you have to take your filter kernel and **flip it before you do correlation**. The alternative way is that you can flip th eimage left right and up and down before you do correlation.

The result would be identical to performing discrete convolution.

**Correlation used for template matching**

**An application of correlation**

Cross-correlation between an image and a prototype and the image patches centered at the local maximum.

<div align=center><img src="https://i.imgur.com/ifnbWQo.png" alt="image-20230214231635662" style="zoom:67%;" /></div>

I apply cross-correlation, and I take out a piece of image and normalise it in an appropriate way. In this way, I can use the this to perform template matching.

Let's assume that we cut out a face from this image. And that face forms my filter kernel. I do a little bit of normalization of the intensities to do cross-correlation in a proper way. I take this filter and then I do discrete correlation.

I would expect that as we move this filter across the image, then, at each location where the pixels in the filter and in the image are similar you get a high value out of this correlation. We get the left image and you can see that there's some extra bright spots and if we take these local maxims, these local bright spots and make a cut out of the same size as the filter. Let's say we have a bright spot and I think that is one guy's face and we then cut out the filter and then plot that as pieces of images that we stick together.

**For template matching, if you have a simple template that you want to search for copies. You can do correlation and find a lot of spots where the correlation filter peaks and then you can select those and say that's where my template matches the most.**

So in this case, we could use this to find the face of a specific person in the crowd.

### Non-linear filtering

#### Definitions

The filter is a **non-linear** function of the input image.

The filter is often **shift invariant**, hence the filter is a non-linear function of the neighborhood window that is independent of position. => We have shifting operation that we slide and neighborhood window across the image and that every location we use the function of the underlying intensities to generate a value for the specific target pixel that we've reached.

It is not possible to define the **filter kernel** as in the linear case.

We cannot use convolution / correlation to define such filters.

#### Example of a non-linear filter - Median / rank filtering

Median filter computes the median of the neighborhood pixels.

The median is the $50\%$ quantile or the rank order that splits the intensities into two equally likely sets.

Rank filtering computes the order statistics of the neighborhood intensity distribution.

<div style="font-family:'Noto Serif SC'">中值滤波（median filter）和秩滤波（rank filtering）是两种基于排序的信号处理方法，它们在某些方面有关系，但也存在一些区别。中值滤波是一种常用的平滑滤波方法，它的基本思想是用信号中某个窗口内的中值来代替该窗口内的信号值。具体地说，对于输入信号中的每个采样点，中值滤波器将该点的数值替换为一个与该点所在窗口中的所有值按大小排序后位于中间位置的值。这个窗口的大小通常是一个奇数，例如3、5、7等。秩滤波是一种更为一般化的排序滤波方法，它不仅可以使用中值作为参考，还可以使用其他百分位数，例如1%、10%、25%、75%、90%等。秩滤波器的基本思想是对于输入信号中的每个采样点，在该点所在的窗口中按大小对所有值进行排序，然后使用指定的百分位数（即所谓的“秩”）来代替该点的数值。因此，可以将中值滤波看作是一种特殊的秩滤波，它使用中位数作为秩。另外，需要注意的是，中值滤波只能对单峰信号进行平滑，而秩滤波则可以对具有任意形状的信号进行滤波。秩滤波是更一般的表达，中值滤波是选取50%的秩滤波。</div>

The discrete median filter / rank filtering

- Sort / rank pixel values in neighborhood in ascending order
- Find the value in the ordered list that at the wanted rank order $q$
- Set the target pixel value to the found rank value

#### Noise removal by non-linear filtering

<div align=center><img src="https://i.imgur.com/bP455gE.png" alt="image-20230214235420981" style="zoom:67%;" /></div>

**Median Filter**

(a) => It looks a little bit like the **mean filter** but it's not computing the mean. There is some smearing of the details, but it doesn't look much different from the mean result.

(b) => It looks much different thatn what we had with the previous mean filter. It looks like it cna clean up the salt and pepper noise. It looks like almost the original or at least very similar to what we have in (a). The reason why we get this is that since we're applying a median filter, even though it's only $3\times 3$ pixels. If we consider a pixel that they contains either salt or pepper, the neighborhood of that which contains a lot of pixels that have the almost same value. => The median of the distribution of those nine pixels in that neighborhood would lead to a gray background pixel. Similar in the case of coin. => That is the reason why median filter does a better job than linear mean filter or linear Gaussian filter.

(c) => It doesn't look like removing the noise, it looks a little bit like the same result we saw previously, the mean and the Gaussian filter.

**Rank Filter**

(a) => It prefer bright values. The coin seems to be a little bit more shiny.

(b) => It has the adverse effect of boosting the noise. It seems to select a bright value. => This filter here is not really doing anything proper.

(c) => If you consider the background white while the background is now uniform. Even though we have the shiny coins, but if the purpose is to reduce the noise in the background, then at least the filter does the job. However, you could still debate whether this is really a good result.

#### Median filtering of different salt and pepper noise levels and filter sizes

<div align=center><img src="https://i.imgur.com/LEOj2mC.png" alt="image-20230215000854927" style="zoom:67%;" /></div>

For the first row ($3 \times 3$) it does a little bit of blurring, but it's not a severe effect. $7\times 7$ gets worse. => Increase the size of the median filter has the advert effect that you are blurring out some details.

For the second row. ($3\times 3$) it seems like generally remove the noise, but one effect that the pixel got overall brighter. But for the $7 \times 7 $, we still have this blurring effect but it looks a bit like that we're getting a slightly better job than $3\times 3$ as the average intensity is more or less the same as in the original case.

For the third row. $3\times 3$ still have a lot salt and pepper noise. 3 by 3 not doing the job. But 7 by 7 fixed it.

Median filtering is a good choice if you have images that are destroyed by salt and pepper noise, but we need to choose the size of the filter carefully. The larger the filter is, the more blurring / smearing of the details you get. But you would need to make a choice depending on how sever the noise is.

### Derivative filters

**More on linear filtering**

#### Image as arrays or functions

How we can use linear filtering to compute the derivatives of the images? That is construct derivative filters.

We had this two views of an image either as an array of pixel values, or we could do this function interpretation. And in the function interpretation, we think of the image as being a function of two variables $x$ and $y$.

<div align=center><img src="https://i.imgur.com/jirdwlq.png" alt="image-20230215010124607" style="zoom:67%;" /></div>

We get this weird landscape. What we realized that an image is just a function or by its discrete representation function.

#### Approximating image derivatives

We could think about how can we compute the derivatives of such a function.

But the porblem is of course that we don't actually have an analytical expression of the image function. We only have a **sampled** version of the image function, so we need to approximate the partial derivatives.

Let's just consider one of them.

<div align=center><img src="https://i.imgur.com/XNH2diH.png" alt="image-20230215011836329" style="zoom:67%;" /></div>

**First row**

When we do the approximation, because $\Delta x \to 0$ in discrete function meaning the smallest step => The next pixel along the row. => I could just move one pixel to the right => I take a step of 1. => Therefore, the approximation.

Even though this is not the limit where we have, but we've just made it as small as possible with the data we have. **So this is one way to approximate a partial derivative on a discrete function.**

This approximation is also known as a forward approximation that is because we're moving 1 step to the right.

We can also have a backward approximation => Midpoint approximation.

**Second row**

We move a row down in the image. We need to look one pixel ahead of the target pixel. => Down

**Higher dimensionality**

Skip

**_We can construct linear filter kernels from these approximations (forward differencing)._**

We just need to subtract neighbor pixels from each other.

Since correlation does not require us to flip the filter, let's start in the case where we want to construct the filter that we can use with correlation.

Let's go with the derivatives with $x$ firstly. It involves a target pixel value $f(x, y)$, and the one that just to the right of that pixel. The operation actually does here is picking out the value $f(x+1, y)$ and picking out the value $f(x, y)$ and changing the sign to $-$. So the filter kernel could look like $[-1, 1]$. We are computing the derivative at $(x, y)$ and we're going to assume that in this filter mask, the $-1$ is the center of the filter. This pixel should be a line with the target pixel $(x, y)$.

Whatever we have at $(x, y)$, we multiply that by $-1$ and then, we take one step to the right. And we take out the value we have at $(x+1, y)$ and just multiply that by 1. At the end, correlation is a sum, so now you have

$$
f(x+1, y) - f(x, y)
$$

So, this implements derivative using the $[-1 \ 1]$ impulse response, and doing correlation. The difference between correlation and convolution is that flipping the filter.

<div align=center><img src="https://i.imgur.com/WwqPPrL.png" alt="image-20230215015059514" style="zoom:67%;" /></div>

#### Other first order derivative filters

We have something called **Roberts** field that it take the derivatives along the diagonal in this 2 by 2 mask. Roberts filter is not really used anymore.

<div align=center><img src="https://i.imgur.com/KUdsfQc.png" alt="image-20230215020100219" style="zoom:67%;" /></div>

Prewitt is doing tomething like a midpoint approximation. It involves a 3 by 3 filter, and requires intensity values in the first column one back in the image. The third is forward in the image. And the target or center of the mask is in the middle. Sobel is similar.

#### Linear seperable filters

Prewitt and Sobel filters are lienar separable.

<div align=center><img src="https://i.imgur.com/TEtvBBf.png" alt="image-20230215020534578" style="zoom:67%;" /></div>

The first row could be viewed as a mean filter. The second row could be viewed as Gaussian filter.

Smearing along the y-axis and computing the derivative along the x-axis.

<div style="page-break-after: always;"></div>

#### Computing derivatives with Sobel filter

Let's consider what the Nobel filter does.

<div align=center><img src="https://i.imgur.com/bjfGE61.png" alt="image-20230215021128840" style="zoom:67%;" /></div>

$derivatives$ are reached by applying the Sobel filter.

If we consider this particular change or edge in the intensities (scarf), we have dark pixels that the values are close to 0 and then the brighter pixel values here.

We are moving from dark to white, from small intensity value to larger intensity value. When we compute the sobel filter, it involves looking 1 pixel ahead (larger) and one behind (small). **That means the filter response becomse positive.**

Once we start to subtract pixels from each other, we can get a negative filter response. So the resulting image could have negative values.

Dark edge is because we are moving from large value to small value. - Dark after rescaling.

**_So the derivatives says about changes in the intensity levels, the contrast changes of the image._**

Computing derivatives is sensitive to noise - we are adding up noise from multiple pixels. => Even though sobel might have sloved it inside, but it still need to use some form of blurring first (Gaussian filter with small $\sigma$). => Then, you can get more robust derivatives that are robust to noise.

Common to smooth image (e.g. with a Gaussian filter) before computing derivatives to be robust to noise.

<div style="page-break-after: always;"></div>

#### Differentiation filters amplifies noise

<div align=center><img src="https://i.imgur.com/EznN3fk.png" alt="image-20230215022513358" style="zoom: 50%;" /></div>

The problem is that for the first derivative, we need to consider two pixel values. We are subtracting together, but if the effect is that we have noise from two pixels being added up, and in the second derivative we have noise from three pixels being added up. That means we'reboosting the noise as we increase the differentiation order.

The way to limit the effect and still get reasonable results is by applying some smoothing filter first such as the Gaussian filter.

#### A snippet of differential geometry

Differential geometry is a handy mathematical formulation for building image analysis algorihms.

<div align=center><img src="https://i.imgur.com/2AOdxvQ.png" alt="image-20230215023411930" style="zoom: 50%;" /></div>

#### Edge detection

An image intensity edge are given by locations in the image of abrupt changes in intensities (high contrast).

Usually detected by

- Extrema of first order derivatives, e.g., maxima of gradient magnitude
- Zero crossings of second order derivatives, e.g., of the Laplacian

#### Zero crossings of Laplacian of Gaussian filter

<div align=center><img src="https://i.imgur.com/6YOFURT.png" alt="image-20230215024415227" style="zoom: 50%;" /></div>

### Filtering color images

Color images and multi-channel images have vector values at each pixel - how do we apply linear / non-linear filtering to such images?

Some possibilities are:

- Apply the same filter to each channel independently.
- Construct a multi-channel filter that computes based on all channels at the same time.
- Change color space, e.g., convert to gray scale or HSV representation and do filtering on a single channel image (e.g. the v-channel from HSV).

What you want to do depends on the task and filter.
