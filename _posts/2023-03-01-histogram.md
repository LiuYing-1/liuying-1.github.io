---
layout: distill
title: Histogram
description: Thresholding segmentation and histogram techniques.
tags: sip
categories: study ucph
giscus_comments: false
related_posts: true
date: 2023-03-01
thumbnail: assets/img/histogram.jpg

authors:
  - name: Ying Liu
    url: "https://di.ku.dk/Ansatte/forskere/?pure=da/persons/762476"
    affiliations:
      name: DIKU, UCPH

toc:
  sidebar: left
---

<div align=center><img src="https://i.imgur.com/BZI3kil.jpeg" alt="1511680791922_.pic" style="zoom:80%;" /></div>

<div class="reference" style="color: 999; font-size: 0.8em; margin-top: 1em; margin-bottom: 1em;">Reference: Lecture from <a href="https://di.ku.dk/english/staff/vip/researchers_image/?pure=en/persons/83684" _target="blank">Kim Steenstrup Pedersen</a></div>

## Pixel-wise Thresholding

In this lecture, we are going to talk about some pixel-wise intensity transformations and we will also be discussing histograms or histograms of intensities and what we can use this for.

### Global thresholding

Using some threshold value $\tau$ we can **construct a binary image** from the input

$$
J(x, y) = \begin{cases}
1, & if \ I(x, y) > \tau \\
0, & Otherwise
\end{cases}
$$

We can use this for a simple **foreground-background segmentation** or segmentation grouping of pixels in the image.

We say that whole bright pixels they now has the value one, they are grouped together. Similar for all dark pixels, all dark pixels they become zero.

<div align=center><img src="https://i.imgur.com/z3NHSe4.png" alt="image-20230301090029439" style="zoom: 25%;" /></div>

Here we piceked a proper $\tau$ and then apply this transformation to the image and then get this binarized image.

However, in the example, some of rice still in the background, that means, the threshold we picked doesn't seem to actually select the individual rice grain so well.

Segmentation algorithm => Group pixels together according to the threshold.

If the scene has a illumination that is varing across the scene, then this simple approach might cause some problem.

But we can fix it => **Adaptive thresholding**

### Adaptive thresholding

Thresholding transformation is non-linear because if you take the intensity and add some new intensity to it and then perform thresholding you get one result.

<div style="font-family: 'Noto Serif SC'">因为如果你把强度加上一些新的强度，然后进行阈值处理，你会得到一个结果。但是如果你把图像在加入一些新的强度之前进行阈值处理，然后对你加入的常数进行阈值处理，那么你得到的结果和你在第一种情况下得到的结果不一样，所以这个是非线性的。</div>

Adaptive thresholding: The idea is that we picked a threshold that depended on the local image.

Now we start to construct non-linear filter.

**Let the threshold depend on intensities in a $N\times M$ neighborhood around the target pixel.**

$$
\tau = mean_{N\times M}(I)+C
$$

$C$ is to make sure that the threshold is high enough, and this $C$ would be the same for all pixels.

<div align=center><img src="https://i.imgur.com/Nn5IdLG.png" alt="image-20230301092513589" style="zoom: 33%;" /></div>

We choose a specific mean filter size, $15\times15$ pixels, $C=20$, then we get this result.

Now we are able to see the rice grains clearly.

It's a non-linear filter because we are operating on a pixels in a non-linear manner. And it is also shifted variant because even though we get different $\tau$ in different location, we are performing the same operation, but the operation now depends on the neighbor pixels.

**_But if I try to apply thresholding approch to more complicated image?_**

I just converted this to grayscale and then it picked some threshold, and then I get this result.

<div align=center><img src="https://i.imgur.com/MsEzgK1.png" alt="image-20230301101202693" style="zoom: 25%;" /></div>

Now this is slightly more difficult to use thresholding. You can't really use this result here.

The problem here is that this intensity distribution of this image is no longer bimodal whereas the intensity distribution for the previous example is bimodal which allows us to pick a proper threshold that separates two groups of pixels.

This way of performing segmentation only works in certain cases where we have a simple distribution of intensities. => **This naive form of segmentation only works for simple images (with bimodal intensity distribution)**

### Range thresholding

**Simple color-based segmentation**

We can also use this approach on a color image.

<div align=center><img src="https://i.imgur.com/5R36F83.png" alt="image-20230301102309064" style="zoom: 25%;" /></div>

**\*Step 1 =>** **Convert image from RGB to HSV.\***

I am interested in segmenting the red picture. I could do this on red channel but it doesn't work. But if I instead pick a range of Hue value, which represents the color.

**_Step 2 => Choose a range of Hue values $[H_{\min}, H*{\max}]$ to represent the relevant color (red).***

**_Step 3 => Set all pixels outside this range to zero and all inside to 1._**

<div style="font-family: 'Noto Serif SC'">例子中显示，我如果对红色通道进行分割，那么无论我怎么选择范围，始终无法覆盖所有红色的水瓶，而且桌子上的橙红色也将被覆盖进去。所以要用 H 通道。</div>

## Intensity Histograms

### Definition

To do some statistics on the intensity of an image, we will first define what we mean by an intensity histogram.

**Intensity histogram**: Count the **number of pixels having specific intensity level** $v\in [0, L-1]$ (assuming $L$ intensity quantization levels).

$$
H_1(v) = \#\{(x, y), I(x, y) = v \},  v\in[0, L-1]
$$

Here the range of $v$ is $[0, L-1]$ is for simplicity, in practice, it should be a set of allowable intensity values.

Basically, the histogram is just to count the number of pixels here denoted by $(x, y)$. We don't care about the coordinates but just the number of pixels that have if its intensity value is $v$.

We can take an image and then, form the histogram of intensities for all possible intensities. => Example

<div align=center><img src="https://i.imgur.com/aw75ZYy.png" alt="image-20230301103805324" style="zoom:25%;" /></div>

In the coins example, we have two peaks. The first peak is most likely the number of the pixels representing the background. The second peak is light gray and this is mostly likely the coins that we observe.

There's better variation in the colors so we have apparently some intensities at most places along the intensity axis.

**_Now we recall from the thresholding on the rice grain image and because the rice grain is looks a little bit like the coin when we do the histogram of the rice grain. We also get a bimodal distribution. There are two modes in the distribution._**

But if we look at the more complex example, then we can see that the intensities distribution is much more complicated so it's hard to find a threshold that separate out the meaningful regions or objects in the image.

If we want to find coins, we probably need to choose a threshold somewhere in the middle of the intensity level and then we might do a good job at segmenting the coins.

### Choosing thresholds

Pick a threshold value $\tau = 126$ and then we're using this, I can do a segmentation and we can see that we get a pretty good result out of this.

<div align=center><img src="https://i.imgur.com/FIVGrxZ.png" alt="image-20230301110544232" style="zoom:25%;" /></div>

The distribution of intensities (gray scale) or colors in an image **reveals information about its content**.

**Bimodal** distribution: Find **an optimal threshold that separates the two peaks**.

This gives a good **threshold-based foreground-background segmentation**.

### Intensity transforms and histograms

Let's just see what happens with the intensity histograms if we apply an intensity transform like the Gamma-correction or exponential and the like.

Firstly, we have some image with the **original histogram** for the image.

Then, we perform a transformation that **takes all pixels and squares them**. Then we can see that in this shape. Especially, we can see the range of the intensities is $10^4$ whereas the original is from $0$ to $255$. So of course we have stretched the allowed intensity range but it also means that we effectively stretch out the intensity. => We would see some holes because we have these white lines where the image there were no observations, no pixels that have the value. So when we do this transformation, we are creating such holes.

<div align=center><img src="https://i.imgur.com/eLrW22X.png" alt="image-20230301111726302" style="zoom:25%;" /></div>

Here is another example with $\log$ transform. It's very clear that at the lower range of intensities, we now get a lot of holes here. Remember that the $\log$ transformation stretching the low-intensity part and compressing the high-intensity part. => Just like the image shown below. However, what we also now see that simply because we have a fixed number of discrete quantization levels, we get these holes (white spaces). We don't have any pixels that have a value in between here. In effect with the quantization level we chose for this image, there will be no pixels that can have this value because that cannot be represented with the quantization level of the original image.

<div align=center><img src="https://i.imgur.com/EsD0rK2.png" alt="image-20230301112320277" style="zoom: 25%;" /></div>

The last one doesn't change the actual shape much. It changes the scale of the intensity axis along the x-axis.

### A probabilistic interpretation

Model choice: The **intensity at any pixel** in an image is a **random variable**.

We can **normalize** the histogram with **respect to the number of pixels** in our image $I$. If $I$ has a spatial resolution of $N\times M$ pixels, then $P_I(v) = \frac{H_1(v)}{NM}$ is the probability that intensity $v$ is found in the image $I$.

<div style="font-family: 'Noto Serif SC'">我们可以这样理解。分子是图片在该强度区间的像素数量，而分母是整个图像的分辨率（面积）。</div>

$\frac{数量}{面积}=概率$

$P_I(v)$ is a **discrete probability mass function (PMF)** (a.k.a. probability mass function $(PDF)$) with $0\leq P_I(v)\leq 1$ and $\sum^{L-1}_{v=0}P_I(v)=1$.

The discrete **cumulative distribution function** **(CDF)** is

$$
F_I(x) = P_I(v\leq x) = \sum_{v\leq x}P_I(v)
$$

### Otsu thresholding

**_How to select an optimal threshold?_**

Consider a histogram with two modes as being separated in two class, then find the optimal threshold $k^*$ by maximizing the between-class variance

$$
\sigma^2_B = \omega_0(\mu_0 - \mu_T)^2 + \omega_1(\mu_1 - \,u_T)^2 = \omega_0\omega_1(\mu_1 - \mu_0)^2
$$

<div align=center><img src="https://i.imgur.com/lhemP8U.png" alt="image-20230301114322459" style="zoom:25%;" /></div>

$$
\begin{align}
k^* &= \arg\max_{0\leq k\leq L-1 }\sigma_B^2(k)\\
\mu_T &= \mu(L-1) \Leftarrow Total\ Mean\\
\mu(k) &= \sum_{i=0}^k ip(i) \Leftarrow Mean\ of\ Class\ 0\\
\omega(k) &= \sum^k_{i=0}p(i)\Leftarrow CDF\\
\sigma^2_B(k) &= \frac{[\mu_T\omega(k)-\mu(k)]^2}{\omega(k)[1-\omega(k)]}
\end{align}
$$

<div align=center><img src="https://i.imgur.com/dfCrHt5.png" alt="image-20230301115425886" style="zoom:25%;" /></div>

## Histogram Matching

**Histogram matching between the histograms of two images**

We consider intensities in an image as a random variable (RV). The normalized histogram gives the distribution of these intensities.

In the following, we will consider limiting case of an image with a continuous range of intensities. That is, we have a continuous probability density function (PDF) over intensities.

<div style="font-family: 'Noto Serif SC'">我们可以先从连续型概率密度函数 PDF 入手，再考虑离散型。</div>

**Question:** Given two intensity probability transforms, can we derive an intensity transformation when applied to one image transforms the CDF (or PDF) to match the CDF (or PDF) of the other image? => If an intensity transformation is applied to the image, can one derive the new intensity probability function? => If an intensity transformation is applied to an image, can one derive the new intensity probability function. **<u>Or, if we have an image with specific PDF, we apply some transformation to the intensities, can we figure out what is the probability density function for this new transformed image?</u>** => **YES**

### Recall - Probability density for continuous RV

<div align=center><img src="https://i.imgur.com/8RBxhDw.png" alt="image-20230301122603178" style="zoom: 25%;" /></div>

### PDFs and functions of random variables (RV)

Let $R$ be a real RV and $g: \R \mapsto \R$ (or $[a, b] \mapsto [c, d]$) a function that maps $R$ to the RV $S = g(R)$.

<div style="font-family: 'Noto Serif SC'">假设我们有一个随机变量 $X$，还有一个用于对 $X$ 做强度变换的方程 $g$。</div>

In the following, I will **<u>use $r$ to denote intensities in the original image</u>** and **<u>$s$ to denote intensities in the transformed image</u>**.

**_Can we compute the CDF $F_S$ from $F_R$?_**

**_Step 1_**. Start from the CDF of $S$

$$
F_S(s) = \int^s_{-\infty}p_S(x')dx' = P(S\leq s) = P(S\in (-\infty, s])
$$

**_Step 2_**. Since $S = g(R)$, we have

$$
S\in (-\infty, s] \Leftrightarrow g(R) \in (-\infty, s) \Leftrightarrow R\in g^{-1}((-\infty, s])
$$

**_Step 3_**. So we can write

$$
F_S(s) = P(S\in (-\infty, S]) = P(R\in g^{-1}((-\infty, s]))
$$

**_Hence, we need to focus and compute $g^{-1}((-\infty, s])$!_**

The simplest case is to **assume that $g$ has an inverse**, which implies that it is strictly monotonic (either increasing or decreasing).

Simplest simple case: $g$ is **strictly increasing**, then

$$
g^{-1}\left((-\infty, s]\right) = (-\infty, g^{-1}(s)]
$$

Thus, the relation between the CDFs are

$$
F_S(s) = P(R \in (-\infty, g^{-1}(s)]) = F_R(g^{-1}(s))
$$

Lets set $r = g^{-1}(s) \Leftrightarrow s = g(r)$, and then, we get

$$
F_S(s) = F_R(g^{-1}(s)) \Leftrightarrow F_S(g(r)) = F_R(r)
$$

If $F_S$ is strictly increasing then we can retrieve $g$ by

$$
g(r) = F_S^{-1}(F_R(r))
$$

**_Explanation_**

Let's consider this scenario that we have an input image.

It has a histogram from the histogram, we can compute the CDF of this image. $F_R$.

<div align=center><img src="https://i.imgur.com/LxgkMOI.png" alt="image-20230301132151016" style="zoom:25%;" /></div>

The formula tells us that if I pick any pixel in my image and if I figure out what I look up what is the intensity value at that pixel. Let's assume that it's $r$.

First, we need to look up what is the value of the CDF of the original image of $r$ => $F_R(r)$.

Then, we need to do an inverse look up in the CDF $F_S$ of the result image that we want.=> Gives me the new intensity value that I should write into the output image. And this value is computing by $g(r) = F_S^{-1}\left(F_R(r)\right)$

**_The consequence is that we can change from the distribution given by CDF $F_R$ to one with CDF $F_S$._**

<div style="font-family: 'Noto Serif SC'">所以，只要我们有了输入图像的 CDF 和想要的输出图像的 CDF，那么我们就可以将强度变换后的图像输出。</div>

We can use this to match intensity histograms of two images. => **Histogram Matching**

**Application**

We can take one image as the input, and compute its CDF. Then, we can take another image which has a distribution that we would like, and then, we compute the CDF of that. Then, we can take the original image and apply the strategy. Construct a version of the image that has the same probability CDF as the latter image.

## Histogram Equilization

**Contrast stretching**

<div align=center><img src="https://i.imgur.com/zu39Wvp.png" alt="image-20230301133522502" style="zoom:25%;" /></div>

We can see there's a lot of pixels in the dark range of the intensity range. Now, the question is can we do something to improve the contrast in this image to stretch the dynamic range of the image so that we can easily see more details in the tire region.

One way to view this is that we would like to make a mapping of this intensity distribution and wrap it towards a probability distribution that is uniform and so that all the intensities are equally.

If we do this, then we sort of redistribute the inside the individual pixel so that they take up ore use the complete range of available intensities for our quantization.

We are looking for remapping to a new PDF with $p_S(s) = \frac{1}{L}$. => Uniform distribution $p_S: [0, L] \to \R$

**_How to find a mapping $g$ such that $S = g(R)$ has a flat PDF?_**

I will go about deriving histogram equalization in a slightly different way than for histogram matching. But it will turn out that we are **_actually performing histogram matching_**.

**_Question_**. We have $p_S(s) = \frac{1}{L}$ and $p_R(r)$, and want to find $g(r)$.

I pull a matehmatical white rabbit and write up this differential equation (we use this notation $g'(r) = dg/dr$)

$$
g'(r) = \frac{p_R(r)}{p_S(g(r))}
$$

The expression above is because => we know that $F_S(g(r)) = F_R(r)$. Lets compute derivative with respect to $r$ on both sides:

<div align=center><img src="https://i.imgur.com/98hGtl3.png" alt="image-20230301142954653" style="zoom:25%;" /></div>

<div style="font-family: 'Noto Serif SC'">用了莱布尼兹和链式法则得到了这个“白兔”。</div>

And since $p_S(s) = 1/L$, we have $g'(r)=L_{p_R}(r).$

Solve this differential equation by integration

$$
g(r) = \int^r_0 g'(\rho)d\rho = L\int^r_0p_R(\rho)d\rho = LF_R(r)
$$

Actually, we have just shown that histogram equalization is histogram matching of CDF $F_R(r)$ to CDF $F_S(s) = s/L$ (i.e. the CDF of the uniform distribution $p_S(s) = 1/L$).

<div align=center><img src="https://i.imgur.com/CQFjl5R.png" alt="image-20230301143626209" style="zoom:25%;" /></div>

### Harsh reality

Perfect histogram equalization is in general not possible due to quantization, missing intensity ranges. =>

<div style="font-family: 'Noto Serif SC'">从图上就可以看出来我们无法得到绝美的结果，因为始终会有 HOLES 由于之间的空白导致。</div>

In practice, not all CDF's are strictly increasing! It is possible to have constant plateau's, if a range of intensities are not occurring in the image.

### Extensions of Histogram Equilization

#### Intensity hisogram equalization of color images

**_How do we perform histogram equalization of color images?_**

**_Step 1_**. Convert RGB image to HSV.

**_Step 2_**. Perform histogram equalization on V channel. V channel is the intensity of the pixel values. Have nothing to do with color.

**_Step 3_**. Convert processed image from HSV back to RGB

<div align=center><img src="https://i.imgur.com/wzKc9es.png" alt="image-20230301145457653" style="zoom:25%;" /></div>

We can see that histogram equalization doesn't do anything good to this region of a bright pixels, it looks a little bit worse than in the original image. But it definitely brightens up the dark part. But we can handle this problem with a variant. => **Adaptive Histogram Equalization / Matching**

#### Adaptive histogram equalization

**A non-linear filter**

<div align=center><img src="https://i.imgur.com/qcY8x1T.png" alt="image-20230301145934595" style="zoom: 33%;" /></div>
