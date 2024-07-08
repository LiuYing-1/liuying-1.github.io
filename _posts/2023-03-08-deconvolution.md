---
layout: distill
title: Deconvolution
description: Image Restoration by Deconvolution
tags: sip
categories: study ucph
giscus_comments: true
related_posts: true
date: 2023-03-08
thumbnail: assets/img/deconvolution.jpg

authors:
  - name: Ying Liu
    url: "https://di.ku.dk/Ansatte/forskere/?pure=da/persons/762476"
    affiliations:
      name: DIKU, UCPH

toc:
  sidebar: left
---



<div class="reference" style="color: #999; font-size: 0.8em; margin-top: 1em; margin-bottom: 1em; ">Reference: Lecture from <a href="https://di.ku.dk/english/staff/vip/researchers_image/?pure=en/persons/83684" _target="blank">Kim Steenstrup Pedersen</a></div>

## Image Restoration by Deconvolution

### Laplacian image sharpening

#### Intuition

In Laplacian image sharpening, we use that the **<u>Laplacian filter ($2^{nd}$ order derivative) has high response values at either side of high constrast changes such as edges</u>**.

<div align=center><img src="https://i.imgur.com/0CTe8El.png" alt="image-20230307133426430" style="zoom: 30%;" data-zoomable/></div>

Here I have a step edge that I have smoothed by convolving with a gassuian of size sigma equals to 10. We just considered the 1D case, but it in general it is 2D.

We have this blue image here, now, if we try to compute derivatives as we've done many times before, we can compute the first derivative and we do this using scale normalized derivative, so I'm multiply by the scale that I computed derivative at here I use the $tau = 10$.

<div align=center><img src="https://i.imgur.com/2ep41DL.png" alt="image-20230307134805724" style="zoom: 30%;" /></div>

<div style="font-family:'noto serif sc'">拉普拉斯锐化图像是根据图像某个像素的周围像素到此像素的突变程度有关，也就是说它的依据是图像像素的变化程度。我们知道，一个函数的一阶微分描述了函数图像是朝哪里变化的，即增长或者降低；而二阶微分描述的则是图像变化的速度，急剧增长下降还是平缓的增长下降。那么据此我们可以猜测出依据二阶微分能够找到图像的色素的过渡程度，例如白色到黑色的过渡就是比较急剧的。或者用官方点的话说：当邻域中心像素灰度低于它所在的领域内其它像素的平均灰度时，此中心像素的灰度应被进一步降低，当邻域中心像素灰度高于它所在的邻域内其它像素的平均灰度时，此中心像素的灰度应被进一步提高，以此实现图像的锐化处理。应用：运用拉普拉斯可以增强图像的细节，找到图像的边缘。但是有时候会把噪音也给增强了，那么可以在锐化前对图像进行平滑处理。</div>

I could have chosen something different from the same value as the sigma but we will just use this. So we get the yellow line. If we want to do **edge detection**, we could look for **local maximum of this first order derivative**, but that is not we are interested here. => _First order derivative is used for edge detection_

If we consider the second derivative, scale normalized, in this case we have to multiply by $\tau^2$, then we get this green curve here.

<div align=center><img src="https://i.imgur.com/5z9I0Rt.png" alt="image-20230307140219382" style="zoom: 30%;" /></div>

Now the idea with Laplacian sharpening is basically that we take the second derivative for images, we consider the Laplacian operator which is the sum of two second derivatives and then, we take the original image in our case, the blue curve, and we subtract this laplacian image (second derivatives) from the blue curve.

We can image that we take the blue curve subtract the green curve, we will be lowering this part because we have higher values of the laplacian in the left side. Similarly, in the right region, we could be rasing values of blue curve and in this way, we can manipulate the blue curve such that it becomes more sharp so the gradient of the curve gets steeper.

<div align=center><img src="https://i.imgur.com/Vf0ZVrJ.png" alt="image-20230307141500386" style="zoom: 30%;" /></div>

The red curve is now that I have taking the original image, the blue curve subtracted the scale normalized second derivative.

$$
Red = Blue - Green \implies (Res = Original - Laplacian)
$$

We can see that if we ignore this bump here, we can see the slope of the red curve is now larger than the blue curve, so we have achieved some form of sharpening.

However, apparently also comes at a cost where we get some **artifacts** out here at this region (red outer parts) and similar down here where the edge starts and ends.

<div style="font-family:'noto serif sc'">观察蓝线和红线，红线作为我们的结果，确实它的梯度变强了因此得到了锐化，但是我们也付出了一些代价，因为红线部分有两个明显突出的假象。</div>

But this gragh is basically the intiituion of what this Laplacian image sharpening algorithm does.

#### General Algorithm

To formulate it in more general for images, we have image $I$, and what we do is we compute the Laplacian response image $\sigma^2\nabla^2L(x, y: \sigma)$ with scale normalization.

We need the mormalization because if we don't do this, the green curve will be very low during the substraction, if we choose a high tau, which would mean that it doesn't really have any effect. So we are in this situation where we actually need to be able to compare an image at two different scales. In this case, it's the original scale of the blue curve which we know here by construction that it's 10 and we compare that with the scale that we use to measure the illustration which in this case was tau = 10. And in order to do this in a fair manner, we need to normalize the derivatives.

**Official Statement**

In general, we can produce a sharpened version of the input image $I(x, y)$ by subtracting the Laplacian response image

$$
J_{sharp}(x, y) = I(x, y) - \sigma^2\nabla^2L(x, y;\sigma)
$$

with $L(x, y; \sigma) = (I * G)(x, y; \sigma)$ and $\nabla^2L = \partial^2L/\partial x^2 + \partial ^2 L/\partial y^2$.

Note: We can use discrete Laplacian filters as described in G&W or the scale normalized Laplacian of Gaussian filter.

The note says we can leave out the scale normalization because then sigma would roughly be equal to 1 except if we use very large filters here.

<div style="font-family:'Noto Serif SC'">如果我们的filters不大，那么一般我们可以直接忽略归一化。默认为 sigma = 1。但是教授还是推荐用后者，归一化的拉普拉斯高斯算子。</div>

Let's look at what happens when we apply this to an image.

#### Implemented using a Laplacian of Gaussian filter with sigma = 1

<div style="font-family:'noto serif sc'">高斯拉普拉斯算子又称为LOG(Laplacian of Gaussian)算子，是在高斯函数的基础上再利用拉普拉斯算子提取边缘得出的一个算子。 拉普拉斯算子是一种高通滤波器，是影像灰度函数在两个垂直方向二阶偏导数之和。</div>

Here I have a blurred image with a Gaussian filter with the sigma equal to 1, and we happen to know the good value for the scaled of the laplacian filter, so we set it equal to sigma = 1. Then, we apply the algorithm.

<div align=center><img src="https://i.imgur.com/necjm6z.png" alt="image-20230307163012880" style="zoom:30%;" /></div>

The third one is the resulting image of the Laplacian with scale normalized.

So we take the original image do the subtraction with the third one, then, we get the sharpened image. => Second

If looking carefully, the scarf and cheek actually look like the edges they are getting more crisp and sharp.

#### Intepretation - Scale Space

Pass

#### Laplacian image sharpening and noise

It enhances noise.

<div align=center><img src="https://i.imgur.com/gk1Fh58.png" alt="image-20230307164849915" style="zoom:30%;" /></div>

Unfortunately, Laplacian image sharpening will have a tendency of boosting any noise that would be in the image.

Here I took again the toy image blurred by gaussian of sigma equal to 1 **<u>and I also added some Gaussian noise</u>** to each of the pixels not much, but enough that you can visually see this. Then we can see that in the Laplacian image, the noise appears to be boosted. This also means that the sharpened version of the original image also becomes noisier.

Basically, what happens is that, the noise accidentally gets enhanced. And we already know why this is.

If you recall from a previous lecture where we discussed the differential filters and the sensitivity to noise. If we have a noisy signal, it doesn't actually have to be that much noise. Then, its result gets amplified or enhanced by taking derivatives and this has to do with the thing that we in fact are subtracting two noisy pixels from each other which is the same thing as adding up noise.

Unfortunately, when the differentiation order increases, this effect gets worse and worse. And now we are considering this a second-order derivative in Laplacian image sharpening. This means that it becomes more dominant than if we had done something that would involve the first derivative. => <font face="Noto Serif SC">拉普拉斯作为二阶比一阶受噪声影响更大。</font>

<div align=center><img src="https://i.imgur.com/Y5FBVUY.png" alt="image-20230307170507179" style="zoom: 25%;" /></div>

### Deconvolution - The Fourier transform approach

In the rest of this lecture, we will consider the problem of image restoration by deconvolution and in fact, there are different approaches to solving this problem. But we will consider approaches that use the Fourier transformation as the way to define the algorithm.

#### Image degradation model

**What problem do we want to solve?**

But first of all, we need to come up with a model of what we mean by degraded image.

So, basically defined, what is the problem?

---

In order to define algorithms for performing **image restoration**, we first need a model of the **image degradation** process.

<div align=center><img src="https://i.imgur.com/Q3Fgrpu.png" alt="image-20230307180557604" style="zoom:50%;" /></div>

So we will consider a system where we have some underlying clean image $f$ that gets degraded by mathematical operator or you can think of it as some system does something to the image.

Then, on top of that, we allow for some noises to be added to the image.

So what we are actually observing is $g$, a degraded image that is undergone some process here.

<div style="font-family:'noto serif sc'">现实中，造成图像退化的种类很多，常见的图像退化模型即点扩散函数（PSF）</div>

[Image Degradation](https://blog.csdn.net/zssyu0416/article/details/80407870)

To be more precise, the general model,

$$
g(x, y) = h[f(x, y)] + n(x, y)
$$

Where $h$ is a **degradation operator** and $n$ is an **additive noise source**.

<div style="font-family:'Noto Serif SC'">我的理解，这就是我们的成像过程：之前说的</div>

$$
I = PSF(F) + N
$$

**_The most important part here is that, I do not observe $f$ directly, I only observe $g$._**

Our problem is then to come up with algorithms that can **restore** $f$ or at least approximate $f$ as close as possible.

**Restoration** produce $\hat f(x, y)$ and is an **inverse problem**. => The restoration procedure is to construct and some approximation $\hat f$ of the nice crisp and clean image here. It is an inverse problem as we only know the degraded image $g$.

---

We have a model of the process that we believe that $g$ has undergone, and then we use this to try to define a solution that gives us a function that is as close as possible to $f$, the nice and crisp image.

#### Linear Shift Invariant (LSI) degradation model

If we assume that the degradation operator $h$ is position independent (shift invariant) and linear, we get the **linear shift invariant degradation model**

$$
g(x, y) = (h * f)(x, y) + n(x, y)
$$

where $h(x, y)$ is the **point spread function (PSF)** of the degradation (imaging) system and as usual \* denotes convolution.

#### Point Spread Function (PSF)

Recall from the first week lectures, what **causes** the PSF in an imaging system:

<div align=center><img src="https://i.imgur.com/0zOy7GI.png" alt="image-20230307184236213" style="zoom: 30%;" /></div>

One of the major causes are that the **aperture** and the **sensor** in the imaging system, they are **finite**. And this will cause **blurring** of the light that hits the actual CCD chip and the actual amount of light that I've recorded on the chip.

On top of it, if we have a imperfect lens, then, it will cause some distortion in the light pattern which is referred to as lens distortion and this also contribute to this PSF function.

Below is an example of a real system.

#### PSF's from real imaging system can be complicated

<div align=center><img src="https://i.imgur.com/B7jMosD.png" alt="image-20230307185423636" style="zoom:25%;" /></div>

So this comes from a microscope that has this PSF function. Let's assume that we have this nice crisp objects here. And we now view this with microscope that has this PSF. That means we need to take this function and convolve with this image. And then we get a result like this.

This example is just to illustrate that you can get some pretty crazy images out of a real imaging system, depending on the complexity of the PSF function. (Up to now, we've mainly just been looking at things like Gaussians and so on)

#### Noise sources

What about the noise? How do we actually model this?

First of all, we made that choice of the noise being additive, you can also define this degradation model in terms of multiplicative noises, but it just involves a little bit more math.

We can model the additive noise term $n(x, y)$ as a field of random variables - one for each pixel in the image.

The random variable in each pixel is governed by a probability distribution and <u>we will often assume it is position independent</u> - in statistics the random variables are said to be **independent and identically distributed (i.i.d.) or spatially uncorrelated.** This type of noise is called white noise (and have a flat power spectrum => There is an equal amount of energy at all frequencies).

If we sample from the noise source, e.g., draw random samples from the field of random variables, we get a noise image $n(x, y)$ which is called a **realization of the noise source**.

Note: it is possible to imagine noise sources that are position dependent (i.e., not i.i.d.) and spatially correlated.

<div style="font-family:'Noto Serif SC'">这在我们的日常生活中也有可能发生，但是这其实只会让我们的模型变得有一点点难而已。</div>

The degradation model is the underlying theory that we need in order to define an algorithm to produce a restored image and remember the restored image is not exactly identical to the clean perfect crisp image, but is an approximation.

#### Examples of common noise distributions

Let's just look at some concrete examples of noises, we've been using Gaussian Noise. But of course we can come up with other forms of noises here like Rayleigh distribution, Gamma distribution, Exponential Distribution, Uniform Distribution and Impulse distribution.

<div align=center><img src="https://i.imgur.com/8z0LgiD.png" alt="image-20230307195831136" style="zoom: 40%;" /></div>

Impulse distribution is basically what the salt and pepper does.

<div align=center><img src="https://i.imgur.com/3qWGtTC.png" alt="image-20230307200047666" style="zoom: 33%;" /></div>

If we try to apply this to a real simplified image with only three different grayscale values. We have added some Gaussian noise, Rayleigh noise, and Gamma noise. It might difficult to see when you look at these specific images here that there's a difference in the noise sources, but once we do histograms of the intensities, we would clearly see there are different distributions in the noise here.

#### Example of image enhancement - Hubble space telescope

This is a concrete example of a real image enhancement or restoration problem. This comes from the NASA Hubble Space Telescope. So originally when this Hubble space telescope launched, it was discovered that the telescopes Wide Field and Planetary Camera had a flawed mirror leading to blurred images.

<div align=center><img src="https://i.imgur.com/18baoGi.png" alt="image-20230308084548752" style="zoom: 33%;" /></div>

To correct this NASA estimated the **PSF** of the telescope and improved the images by **deconvolution** using the estimated PSF.

Later the error was corrected by adding an extra mirror.

<div align=center><img src="https://i.imgur.com/67V8gGm.png" alt="image-20230308084828337" style="zoom: 33%;" /></div>

This is a star in the middle and an image taking of this star with this defect in the mirror. You could bascially see that this the sort of like a weird ringing pattern and that's actually the PSF function that we're seeing right there.We can think of stars as being point light sources in this setup here. So it would correspond to convolving with a PSF on images where you had a bunch of deltas spikes.

The problem here is that you have several spikes or point light sources. But anyway, we can do things that try to estimate a function of PSF for this flawed mirror.

<div align=center><img src="https://i.imgur.com/FGIwSDs.png" alt="image-20230308085616164" style="zoom: 33%;" /></div>

The right one is actually not the result of deconvolution using this estimated PSF but the result of the improved mirror.

But the idea is sort of the same, we would like to have an algorithm that could take an image like before, and generate that which looks nice one.

#### Direct Inverse Filtering

So the image restoration process goes like this.

We have this underlying degradation model which involves some clean version of the image $f$ that has been convolved by the system PSF, and some noise has been added and then, we observe this $g$. This is what we get out of our imaging system.

Our problem is to come up with some filter that takes $g$ as its input.

It should take $g$, the degraded image. It should take some estimated least of the system $h$, and some statistics of the noise, maybe even the probability distribution of the noise. Put these things together, what we would like is to solve this inverse problem of finding $f$ or rather at least get an estimated or approximation of $f$.

<div align=center><img src="https://i.imgur.com/vZqB9Ez.png" alt="image-20230308101528791" style="zoom:33%;" /></div>

This is the type of algorithms that we're looking for some form of filter and is most likely a non-linear filter.

**_OK, let's start with the simplest thing we can do in order to restore an image here._**

This process here will be called **_deconvolution_** because it is similar to sort of inverting the process of convolution, and it is also called **_direct inverse filtering_**.

The idea is that we start from the model, the LSI degradation model is

$$
g(x, y) = (h*f)(x, y) + n(x, y)
$$

Then we take the Fourier transform of the model, is (**multiplication**)

$$
G(u, v) = H(u, v)F(u, v) + N(u, v)
$$

So we have a convolution here in space, then, we know from the convolution theorem that the product in frequency domain. So it means that we need to take the Fourier transform of the $h$, and the Fourier transform of the clean image $f$. Similar to the noise, we just take the Fourier transofrm of an noise and add that => $N$.

So this is the Fourier transform of the degraded image.

---

**_Note:_** _Just as what we have mentioned before, we could imagine a multiplicative noise model where instead of a plus but also a multiple application here. But that would make this Fourier Transform a little bit more involved because if you had a multipilication in space, that would translate into a convolution frequency which would mean that you would bascially have to take the term $H$ and convolve with the Fourier transform of the noise. => So becomes a little bit more involved._ We only consider the simple case here.

---

If we also make it even simple and assume there is no noise, so,

In the noise free case, $n(x, y) = 0$, deconvolution can be performed by the **direct inverse filter**.

<div align=center><img src="https://i.imgur.com/lU4cpBu.png" alt="image-20230308104525667" style="zoom:25%;" /></div>

If these several assumptions hold, this is actually the exact solution.

Let me just add a little bit of extra thing.

Assumes we know the Fourier transform of the PSF $H(u, v)$, also known as the Optical Transfer Function (OTF).

Aso assumes that $H^{-1}(u, v)$ exists. In this case, the direct inverse filter provides an exact solution.

<div align=center><img src="https://i.imgur.com/gCQOSAM.png" alt="image-20230308105027817" style="zoom:33%;" /></div>

Any OTF where for some frequencies $(u, v), H(u, v) = 0$, is non-invertible (Division cannot be $0$). This problem can in some instances be handled by adding a small constant to $H(u, v)$ at all frequencies. Now the solution is only an approximation to the original (un-degraded) signal.

So once we know the $H$ function, this is how we get an estimate of the Fourier transform of the clean image. So you just need to do the inverse Fourier transform, you get the clean image back.

Let's look at an example.

Here we have a blurred image with Gaussian PSF $2^{nd}$. The corresponding OTF for $h$ is $3^{rd}$.

<div align=center><img src="https://i.imgur.com/1j3I1qq.png" alt="image-20230308110035329" style="zoom:33%;" /></div>

So, we take the degraded image, divided by the OTF, do the inverse Fourier transform of that result, and then, we get the clean image back.

#### Deconvolution in the presense of noise

But, what happens if we introduce noise?

What happens in the noisy situation, $n(x, y)\neq 0$?

$$
\hat {F} = F(u, v) + N(u, v)/ H(u, v)
$$

**The noise is boosted** by the direct inverse filter and will dominate the output solution.

Example: If the noise have a flat constant power spectrum (white noise), then for all requencies where $H$ is small, $H << N$, the ratio will be large. These frequencies will dominate the output.

<div style="font-family:'Noto Serif SC'">观察第二项，如果我们在傅立叶域在某点有很小的H，这样就会导致噪声特别大。</div>

Example here a blurred image with some noise. And we can see the result is not so well by using direct inverse filtering approach.

It could be like this is because our OTF here is close to zero for a lot of frequencies.

#### Problems with noise

<div align=center><img src="https://i.imgur.com/slL6gfJ.png" alt="image-20230308111042559" style="zoom:50%;" /></div>

#### The Wiener filter

How to fix the problem of the direct inverse filter had **in the presense of noise**? => Wiener Filter

The idea is to find another filter $\hat H$ that can be used to deconvolve the degraded image as close to the original image as possible, or $\hat F(u, v) = G(u, v)/\hat H(u, v)$.

Solution to the least squares minimization problem

$$
\min_{\hat f} E\{ (\hat f - f)^2 \}
$$

where E is the expected value and $\hat f$ is recovered image and $f$ is the unknown clean image.

The optimal LSI inverse filter in the least squares sense is called the **Wiener filter**.

The solution has the inverse optical transfer function (OTF).

$$
\frac{1}{H'(u, v)} = \frac{1}{H(u, v)} \frac{|H(u, v)^2|}{|H(u, v)|^2 + S_n(u, v)/S_f(u, v)}
$$

Where

$$
S_n(u, v) = |N(u, v)|^2, S_f(u, v) = |F(u, v)|^2
$$

are the power spectra of the noise and the original image.

Remember that

$$
|X(u, v)|^2 = X(u, v)\bar{X}(u, v)
$$

with $\bar{X}(u, v)$ denoting complex conjugation.

#### Accounting for noise

The ratio $S_n(u, v)/ S_f(u, v)$, averaged over all frequencies $(u, v)$, is the inverse of the Signal-to-noise-ration (SNR).

<div style="font-family:'Noto Serif SC'">信噪比，how big is the noise relative to the actual signal.</div>

For white noise, the noise spectrum is a constant.

The power spectrum of the un-degraded image is seldomly known, but we may approximate the ratio as a constant.

The Wiener filter then reduces to

$$
\frac{1}{H'(u, v)} = \frac{1}{H(u, v)}\frac{|H(u, v)|^2}{|H(u, v)|^2 + K}
$$

Where $K$ is a scalar parameter.

We only need to know the Fourier transform of the PSF, so we need to have an estimate of the PSF function of our imaging system.

Below is the example of the Wiener Filter

#### The constant approximation is usually good

Let's look at the effect of using this constant approximation.

<div align=center><img src="https://i.imgur.com/qDpZaFW.png" alt="image-20230308121511649" style="zoom:50%;" /></div>

Here we have a noisy blurred image, this is what happens when we use this constant approximation of the signal to noise ratio. The third one is the power spectrum of th enoise and the power spectrum of the clean image.

You can see, if you compar these two deconvolved images, this image area is slightly more blurred than this. Of course, we don't get rid of the noise, that's not the purpose of the Wiener Filter. It's just trying to remove the effect of the convolution that happened in the imaging process. (Remove the effect of the PSF).

It might look slightly better when we actually know only all things so we can actually compute the correct signal to noise ratio. But the second one is already enough.

So, in general, it's the one in the middlt that you would use this as the third one does not really happen often in practice.

#### Comparison

Let's compare direct inverse filtering and Wiener Filtering.

<div align=center><img src="https://i.imgur.com/g4aaNXX.png" alt="image-20230308124329240" style="zoom:33%;" /></div>

Assume that we have motion blur so that means that the PSF is all dragging or everythiing along some direction where we have motion. And we have added a little bit of noise.

#### Origin of the Wiener filter

The Wiener filter is the solution to a regularized least squares problem.

And the problem looks like this, so it's defined by a functional if you're into machine learning, you can also think of this as a lost function where we want to minimize for $f$ that is the clean or at least an approximation of the clean image.

$$
E[f] = ||f-g||^2 + \lambda||f||^2
$$

We have this expression of that involved both $f$ and degraded image $g$.

So, $f$ is what we want, and we don't know it yet. But $g$ is given by observation.

The expression means that we are comparing or taking differences squared differences between two function here or in the discrete case. The first item you can think of it to be the pixel-wise difference. And then, square this and take the sum overall pixels.

The second term is the regularization term. We should take the pixel-wise squared and take the sum of all them.

We are looking for the solution should be smooth and in this case the squared value should not be very large. The $\lambda$ allows us to control how much the smoothness term should dominate compared to this data term.

We need to find $f$ to make the minimum of this expression.

#### Alternative view

There's also an alternative view on which might also give a little bit of inside and that's we can actually put a probabilistic interpretation of this. The Wiener filter is the maximum posteriori (MAP) estimate under the assumption of Gaussian noise and a Gaussian white noise prior distribution for unkonwn $f$ (a prior probability distribution on the space of images)
