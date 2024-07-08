---
layout: distill
title: Convolution
description: Pixel-wise Operations, Intensity, Transformations, Image Formation, and the Convolution Integral.
tags: sip
categories: study ucph
giscus_comments: true
related_posts: true
date: 2023-02-12
thumbnail: assets/img/convolution.jpg

authors:
  - name: Ying Liu
    url: "https://di.ku.dk/Ansatte/forskere/?pure=da/persons/762476"
    affiliations:
      name: DIKU, UCPH

toc:
  sidebar: left
---

<div align=center><img src="https://i.imgur.com/k1XNh2S.jpg" style="zoom: 80%"></div>


<div class="reference" style="color: #999; font-size: 0.8em; margin-top: 1em; margin-bottom: 1em;">Reference: Lecture from <a href="https://di.ku.dk/english/staff/vip/researchers_image/?pure=en/persons/83684" _target="blank">Kim Steenstrup Pedersen</a></div>

## Pixel-wise Operations

### Image file formats and compression

Here is just an overview of various file formats to give you an idea of how we can store and read the images. There are many different file formats that have made both storing on disk and transferring images on the web.

It depends on the application area and which format you use. In this course, we are going to use the following sets of formats.

**JPEG**

- Provides **<u>lossy compression</u>** and can store pixel data in the **<u>quantization of 8 bit unsigned integer per color channel (RGB)</u>**. Compression level **<u>is controlled by a quality parameter</u>**.

**PNG**

- Provides **<u>lossless compression</u>**. Can store pixel data in the quantization of 1, 2, 4, 8, or 16 bit unsigned integer per channel and supports pixel formats grayscale, RGB, indexed, grayscale and alpha, and RGBA (A=alpha channel). ***Alpha channel*** is the channel for transparency. 

- <div style="font-family: 'Noto Serif SC'">阿尔法通道（α Channel或Alpha Channel）是指一张图片的透明和半透明度。例如：一个使用每个像素 16 比特存储的位图，对于图形中的每一个像素而言，可能以 5 个比特表示红色，5 个比特表示绿色，5 个比特表示蓝色，最后一个比特是阿尔法。在这种情况下，它要么表示透明要么不是，因为阿尔法比特只有0或1两种不同表示的可能性。</div>

**TIFF**

- Provides **<u>lossless compression</u>** and allows for **<u>no compression</u>** as well. Can **<u>store pixel data in quantization of 1 to 32 bit unsigned integer per channel</u>** and supports pixel formats bilevel, grayscale, indexed, RGB and others. Allows to **<u>store multiple images</u>** in one file. Share raw data between cameras.

### Lossless versus lossy compression

<div align=center><img src="https://i.imgur.com/hJssMnS.png" style="zoom:67%;" /></div>

Lossless compression preserves all information and the process is invertable. That means, we can decompress and get exactly the same as input to the compression algorithm. Lossy compression is done by approximating the image (throwing away information). That means you cannot invert the process and get the same image back as you put into the compression algorithm. But lossy compression usually leads to much more higher compression factor so more compact representations than lossless compression does. 

<div style="font-family: 'Noto Serif SC'">表示更紧凑</div>

Compression is a huge topic but out of scope for this course.

### Lossy image compression creates artifacts

**Can be devastating to image processing and analysis.**

What is the consequence with the JPEG compression, especially when we have very low quality representation (a low quality parameter), the point is that it has a potentially devastating effect. Below is an example.

<div align=center><img src="https://i.imgur.com/cYEAicX.png" style="zoom: 33%;" /></div>

This image is stored in JPEG, and the quality parameter was chosen very low. So the file size is low, but the quality of the image is pretty bad. If we look carefully, we might see more blocking effects up in the sky, like patches of sky where it seems that you have a constant color, and you can see sharp edges between them.

If we start to do some manipulation, I'm just taking the RGB image converting it to the Hue representation color space. And then, I just show the Hue channel. Remember the Hue channel is just one value per pixel thing. I here used matplotlib *HUE*.

It is clear that there's something come completely wrong with this image. We see fake structures these checkerboards so we can get misleading results. So in general, <u>we should be careful using lossy compression when we want to do image processing and analysis, because you have this effect that you are throwing away information. Especially if you use a low quality parameter when you do this JPEG compression.</u>

## Basic pixel-wise operations

### Image blending - simulate motion blur

<div align=center><img src="https://i.imgur.com/BztlJ2b.png" style="zoom:25%;" /></div>

Simple mathematcial operations that we want to do on images.

Pixel-wise operations, this means, it's a mathematical operation that I apply to every single pixel in an image.

An example of what we might want to do are 

- Adding or subtracting a constant
  
  
  $$
  J(r, c) = I_A(r, c) + C
  $$
  
  
  I take some input image $I_A$, and add pixel given at row $r$, and column $c$ and add some constant $C$. At any other pixel, I do the same thing, adding a constant, then, I get a result image where it's the original image where every pixel has been shifted by this constant here. You should figure it as we don't really change the spatial structure or the pixels. We only change wahtever values you have inside the stored in it.
  
- Multiplication or division by a constant
  
  
  $$
  J(r, c) = C\cdot  I_A(r, c)
  $$
  
  
  It's pixel-wise. We multiply by this constant $C$ at every pixel $(r, c)$. That returns a new image $J$.
  
- Adding or subtracting images (pixel-wise)
  
  
  $$
  J(r, c) = I_A(r, c)+I_B(r, c)
  $$
  
  
  We can incolve several images. For instance, we can take two images, $I_A$ and $I_B$ and add them together. Pixel-wise add or subtract if you want, that means, for a specific pixel at row $r$ and column $c$, we take whatever value we have stored in the image $A$ and add it to whatever value we have stored in the image $B$ and that gives you a new resulting pixel value that we store in this image $J$ at the pixel $(r, c)$.
  
- Multiplication or division of images (pixel-wise)
  
  
  $$
  J(r, c) = I_A(r, c)\cdot I_B(r, c)
  $$

All operations here are done per pixel.

! ***Notice***: When we do this, we need to be careful and think about the quantization levels that we have chosen for our image representation. We can easily get either over- or underflow as we do see these operations. So, normally, we need to combine such operations (pixel-wise) with some clippling of the range of output. 

For example, you choose an 8-bit representation for every pixel. And it's an 8-bit unsigned pixel. If I do the first adding operation, add a constant that make a new resulting pixel value in $J$ larger than what we can represent in 8-bit unsigned integer so that is larger than 255. It would have overflow.

### Reminder - Quantization of samples/pixels/voxels

We have the quantization levels depending on what type of images we store. You might have this problem with under and overflow quicker than in other forms. If you think in terms of 8-bit unsigned integer, then you would very fastly run into this problem. Binary is even worse.

### Images in Python and Scikit-Image

***How do we do this manipulations in Python and using this Scikit-Image pack?***

 The first thing to notice is that Scikit-Image uses the numpy arrays as the representation of the image. So once you've loaded an image in by Scikit-Image, what you get returned is a numpy array. `numpy.ndarrays`

That means you can all the operations that you can normally do on numpy arrays, you can also do that on this kind of image arrays. They are the same data structure.

- `numpy.ndarrays`

- For-loops are slow in Python, so a more efficient way of applying pixel-wise operations is to **<u>use numpy's functionality of broadcasting</u>**.

  - Here is an example of adding a constant to all pixels

    ``` py
    from skimage.io import imread
    I = imread('myimage.jpg')
    J = I + 50 # Adds 50 to all pixels in I
    ```

  - **Both `I` and `J` are `numpy.ndarrays` of the same dtype and shape.**

  - The data type of every pixel is stored in the attribute called dtype.

+ Be careful what dtype your image has - when manipulating pixel values overflow and underflow may occur.

+ Handle this by clipping and type casting: => Method 1

  ``` python
  from skimage.io import imread
  I = imread('myimage.jpg') #dtype = unit8
  J = np.clip(I.astype('int16') + 50, 0, 255).astype('uint8')
  ```

  I changed the dtype of it and you can do this with `astype` method. I change it to signed 16 bit integer, and now I add 50. And the result of this is an numpy array that has the same shape as `I`. But now has the dtype in 16 bit. The idea is that make sure that I pick a data side that can actually represent the result of this edition.

  But what I want is actually not something that represented in 16 bits. I want to make sure that the resulting image $J$ is get converted back to unsigned integer 8 bit integer something that I might be able to write down to a new JPEG file. The way to do that is to use the built-in function in numpy function which is called `clip`.

  It takes some array that's given by the result of this evaluating this expression. And it also takes a range of values that I want to ensure that all elements of this result that they are clipped or put into this range. So here I say that whatever the values are here, they should fit in the range $0\to 255$. So, if I have a value that's bigger than 255, this value gets changed into 255. If I have a value that's less than 0. So it's a negative number, then it gets changed into 0. It doesn't do any clever scaling or anything it just cuts off values that are outside this range them into the limits of this range. Once I have done that, it's safe to convert this resulting array back ot 8 bit unsigned integer because we know that it can be stored in an unsigned 8 bit integer. So I use again this function to convert it back. So the result is that $J$ has the same shape as $I$, and has the same dtype.

+ Or switch dtype, to floating point representation:

  ``` python
  from skimage.io import imread
  from skimage import img_as_float
  I = img_as_float(imread('myimage.jpg'))
  J = I + 50 # Now both I and J has dtype = float
  ```

  Start by switching my dtype to **<u>floating points representation</u>**. After the operation, convert it back by using clipping trick back and convert to 8 bit before I can save it into something like JPEG image.

### File formats for float images

If I want to save the resulting image as a floating point image, I cannot use the standard image formats like PNG, JPEG and TIFF, and so on. I need to use some other format. Here aer some suggestions of the numpy as your way to save the floating point images.

Now there are two types of files. You can save with the .npy (one array) and .npz (several arrays). => .npz is the file format that you can save multiple images in one file.

- `numpy.save` saves a numpy array to disk in a file (.npy).
- `numpy.savez` saves several numpy arrays to disk in a file. (.npz)
- `nump.load` loads the content of `.npy` and `.npz` files.

### Change the contrast to improve visibility

<div align=center><img src="https://i.imgur.com/W7lFGHg.png" style="zoom: 25%;" ></div>

Some effects of performing some of these basic operations to all pixels that we have discussed.

So the basic operations like this could think of that as a very simple way of doing contrast change. We're going to talk its essence use basic operations on each pixel.

So we build up more complicated algorithms for instance, for enhancement based on operations like these.

### Pixel-wise intensity transformations

#### Image negative

I took the a grayscale image and I want to produce the image in the right part.

<div align=center><img src="https://i.imgur.com/7UXn3Z4.png" alt="image-20230212205444105" style="zoom: 25%" /></div>

This is by a function $g$ that I want to apply to every pixel. I want to use pixel-wise.

If we have an image like this, it's an 8 bit unsigned integer representation. But that's no guarantee that the range of these pixel values are between $0$ and $255$. So if I take into account what the actual effective range of pixel values are, I get a nice looking result image.

So if I look through all of the pixels and record just the smallest intensive value. In this case is 25, and the maximum intensive value is 245. Then I can construct the nice looking image here by choosing the function $s = g(r)=I_{max} + I_{min} - r$ where I compensate for this range by **<u>adding maximum and minimum and then subtracting whatever pixel value I have in the input image</u>**.

The important part is that this shows you that I can make a function that I can apply to every single pixel, so it's applied pixel-wise.

The result images that for every pixel $(x, y)$, I apply the $g$ function and then, I get a new pixel value that I store in my new image $J$ at the corresponding pixel location. I do that for all pixels.

In Python and using numpy, you can do this. You just have to define a function that represents $g$ and then takes one variable as its input $r$ and then this can be applied to all pixels by numpy broadcasting.

**Subsummary in this part**

The **Dynamic range** is the actual range of intensity values $[I_{min}, I_{max}]$ present in the image. This does not have to be identical to the quantization limits.

We can change the **dynamic range** of an image by applying **function** $g$ on intensity values.

In general, we can apply some function $g$ to the intensities pixel-wise

$$
J(x, y) = g(I(x, y))
$$

#### Logarithmic transform of intensities

Compress the dynamic range

$$
J (x, y) = c \ln [1 + (e^\alpha - 1)I(x, y)], \alpha > 0.
$$


Scale to output intensity range, e.g., 


$$
c = \frac{255}{\ln [1 + \max (I(x, y))]}
$$


Stretch the lower intensities (dark) and compress the higher intensities (bright).

<div align=center><img src="https://i.imgur.com/EEN9TPA.png" alt="image-20230212211826450" style="zoom: 20%;" /><img src="/Users/liuying/Library/Application Support/typora-user-images/image-20230212211955176.png" alt="image-20230212211955176" style="zoom: 20%;" /></div>

Use if you have an image with bright and dark regions and you are interested in the details in the dark regions.

#### Exponential transform of intensities

Stretch the dynamic range
$$
J (x, y) = c[(1+\alpha)^{I(x, y)} - 1], \alpha > 0.
$$
The opposite of lograithmic transform: Compress the dark pixels and stretch the bright pixels.

<div align=center><img src="https://i.imgur.com/1AKzXDG.png" alt="image-20230212212240921" style="zoom:25%;" /><img src="https://i.imgur.com/IBvY89f.png" alt="image-20230212212419752" style="zoom:25%;" /></div>



#### Power-law (Gamma) transform

Compression and stretching of dynamic range


$$
J(x, y) = c[I(x, y)]^\gamma, \gamma > 0
$$


and $c$ chosen to map input dynamic range full output range.

<div align=center><img src="https://i.imgur.com/vzcgtVe.png" alt="image-20230212212637530" style="zoom:25%;" /><img src="https://i.imgur.com/vYQSRhz.png" alt="image-20230212212705093" style="zoom:25%;" /></div>

Some other applications of gamma-correction:

- Improve the raw signal from the CCD chip

- Enhance intensities to fit display hardware


$$
J(x, y) = h(g(I(x, y)))
$$

#### Gamma correction on a color image

<div align=center><img src="https://i.imgur.com/jUHRhVM.png" alt="image-20230212213001714" style="zoom:25%;" /></div>

#### Effect of degrading the intensity quantization

<div align=center><img src="https://i.imgur.com/D6iH1HB.png" alt="image-20230212213051704" style="zoom:25%;" /></div>

Let's go back to quantization level because that's also a little bit of extra information that we can get from the resolution that we have in quantization. How many quantization do we have?

In the previous example, we see that lowering the quatization, we will see the effect of that we lost details in the resolution. But we can also study the effect of each of the bits that we use to represent the intensity values.

#### Bit-plane slicing

If we consider the concept of a bit-plane.

If we think of an image as at every pixel we have 8 bits to store the value. Then we can consider the least signifincant bit first. Let's slice off all the remaining bits and only consider the least significant bit. If we do that for all pixels, then you could think of this as a binary image. => Bit-plane.

I can do this for any of the bits in our representation all the way up to the most significant bit.

<div align=center><img src="https://i.imgur.com/GeDfW5H.png" alt="image-20230212213829828" style="zoom:25%;" /></div>

So slicing off the least significant bit means that I take only the first bits for each pixel.

An image represented with a byte in each pixel (e.g., grayscale images) maybe decomposed into the individual bit-planes => This is called bit-plane slicing.

Bit-planes can be extracted using the bitwise `AND` operator on the pixel and a bit mask for the bit-plane. E.g., use `numpy.bitwise_and()`.

Useful in image coding and compression. => Applications Example below.

<div align=center><img src="https://i.imgur.com/5Etnbpq.png" alt="image-20230212214309049" style="zoom:25%;" /></div>

We apply a bit mask that selects each of the bits so this is a binary image and it shows what is stored in the least significant bit (first). All the way up to the most significant bit.

**<u>It's fairly difficult to see any structure in the least significant bit, and the second and the third. So that means the number changes in the actual numbers that are stored in the pixels. That this doesn't carry much information about what is actually in the image.</u>**

So if you consider the most significant bit, it looks like the most significant bit can see sort of the large scale structures of the image. We can actually see the face but we don't have like minor details like in the texture and the hiar and so on. But we have an idea of what this is stored.

**<u>Now this gives you an idea of how you can actually do compression because what we might want to store is not the least significant bits. So in this case, we would actually throw away the first second and third bit and only save the last five bits.</u>** And this would be a lossy compression algorithm.

#### Application

<div align=center><img src="https://i.imgur.com/6vtP1BL.png" alt="image-20230212215540544" style="zoom:20%;" /></div>

**Keeping only the 5 most significant bits. Bits 1~3 is set to zero in the reconstruction.**

It's difficult to see the changes and we can actually illustrate what the differences between the original and this compressed image by subtracting this image and htis image from each other in a pixel wise manner. So that means I take image one subtract image 2. And then, I can see the difference image. It's not a binary image, but an image where I have values that are stored in the least significant bit from 1 to 3 bit. It's actually a grayscale image.

It is the fact that we can see some details in the difference image, but mostly are some rubbish that we can throw that away.

And here we get the lossy compression algorithm that thrown away the least information.

## Image formation

### A Mathematical Model of Image Formation

Now, we are going to develop a mathematical model for a image formation.

Here means **<u>the process of taking a picture with our camera</u>**.

So that model will be developed using an optical imaging system as the model as driving the example. But the model can be used for any imaging device whether this is a telescope or normal camera with microscope, CT scanner or a mask scanner or whatelse we could come up with.

So let's start out with defining what is the optical imaging system.

We already talked about obscure / pinhole camera.

I showed you this illustration before you have light out in the real world that goes into the camera by a pinhole. We will refer to this as the **<u>aperture</u>** of our imaging device. In the optical case, we have all light travels through straight line goes through the pinhole and it goes into the camera and hits the back side of the camera. This back side will call the **<u>image plane</u>**. We have information out in 3D that gets predicted into this image plane. This is known as a **<u>perspective projection</u>**.

You can have all the types of cameras where you don't have this perspective projection. => ***Orthogonal Projection***

But in this course, we only talk about perspective projection and this is an idealized view of what is going on when we take pictures with a camera.

In the real camera, the pinhole is of some finite size and it's not just a point. And infront of it, we would usually have some lens that collect the light and focuses on the pinhole.

<div align=center><img src="https://i.imgur.com/PqoHcld.png" alt="image-20230213000948251" style="zoom:25%;" /></div>

#### Perspective projection

Perspective projection causes:

- Foreshortening - the apparent size of objects depends on distance to viewer.
- Convergence - Parallel lines meet in vanishing points. 

There are a lot of extra things to go with perspective projection. => Advanced courses for reconstructing 3D points what we see via camera.

#### A snippet of computer vision

There is a mathematical model that describes what the pinhole camera does this idealized camera.

The pinhole camera performs a perspective projection onto the image plane.

<div align=center><img src="https://i.imgur.com/JlL3XRy.png" alt="image-20230213002803981" style="zoom:25%;" /></div>

We have a point out here in the real world which could be described by a 3D coordinate. Let's call it $P$ and the coordinates are given by this triplet. It sends out some light and we see it in the camera by the light that travels in the straight line through the pinhole and continuously unit it hits the image plane.

So in reality, you could think of it to find the point in the image plane and you need to form this line here that goes from this 3D point where the pinhole is in 3D and then trace the line unit it hits the image plane.

The 2D coordinates in the image plane is $p$. The $Z$ parameter in original 3D is replaced by the parameter $\frac{f}{Z}$ which is transformed into 2D projection.

Generally, the pinhole is called the optical center, and the distance between the image plane and the optical center is called the focal length $f$ and this is the property of the camera which is fixed.

So this just tells us about how we can map a 3D point onto the image plane so onto a pixel coordinate in the image. => Ordinary camera

#### Some observations

What we need right now is a model of what happens light travels from out in the real world through potentially some **optics** through the **aperture pinhole** and onto the **image plane**.

<u>First of all, any optical system (camera with an optical lens) that you might put in front of the camera will distort and smear / blur the resulting image (with varying degree depending on the quality of the opticals).</u>

The <u>size of the pinhole / aperture</u> has a smearing / blurring effect. So the larger the hole is, the light goes through the more smeared out the image might actually appear.

A digital sensor (e.g., CCD chip) will smear / blur the image a bit due to the fact that <u>sensor cells are not infinitesimally small</u>.

***If we want to form a mathematical model of an imaging system, we need to consider the effect of three factors above.*** And this will end up with a very central concept in signal processing called **<u>convolution</u>**.

Convolution is a processing tool that performs changes to an image and generates new image. We can also use it as the machinery in image analysis.

### A mathematical model

We want to model the process of forming an image with our imaging system. => A camera

Consider an imaging device as a system that maps input **distributions of light** (in general electromagnetic radiation) to output **light distributions** on the image plane. => How much light do we have in different areas in front of the camera and how does that gets mapped onto the image plane.

We will represent the imaging system with an operator $S$ acting on the input distribution $I$, $O=S\{I\}$.

<div align=center><img src="https://i.imgur.com/xzpoMVR.png" alt="image-20230213010543381" style="zoom: 25%"/></div>

We have some input light distribution $I$ which in reality is some distribution in 3D world in front of us, then, the camera applies some operator $S$ on the input distribution and generates as output at distribution on the image plane.

#### Imaging systems as a linear system

##### First assumption

We will for simplicity use a linear system as model. We think of it as a function that maps the distribution to another distribution.

<u>An operator $S$ is linear, if $S\{aX+bY\}=aS\{X\} + bS\{Y\}$.</u> 

This means, if I have two different light distributions $X$ and $Y$, two constants $a$ and $b$. Then, applying the imaging system by the operator $S$ to distribution $x$ and the distribution $Y$ Separately. Multiplying them by some constants $a$ and $b$ and adding together. This should give the same result as if I took the $x, y$ distributions multiplied by some constant $a$ and $b$ and added them together. Finally, applied my camera $S$ operator.

So, this equality must hold for is to be linear. Illustrative example is below.

<div align=center><img src="https://i.imgur.com/DdCwpvG.png" alt="image-20230213011957243" style="zoom:25%;" /></div>

##### Second assumption

Open up the black box of $S$ and try to be a bit more precise.

<div align=center><img src="https://i.imgur.com/JOea4fD.png" alt="image-20230213012518127" style="zoom:25%;" /></div>

Let's try to model the camera in this way.

We're goingto assume that the camera contains two components, PSF and some noise. => $S\{I\}=PSF\{I\} + noise$.

$PSF$ is the Point Spread Function and it must be a linear operator, otherwise, our $S$ will not be linear.

Noise - will model the non-deterministic part of the imaging system.

The process is that camera took the light source as an input, and then apply $S$ so that our resulting image is the output which has been smeared out.

##### Sources of noise in image formation systems

**Common sources of noise.**

As we take pictures, there are vaiours sources of noise and all depends on the type of camear or imaging system that you have.

**Capture** 

+ It could be for instance, if you take an ordinary optical camera and take pictures, then there might be some fluctuations in the actual amount of light that you see out in the scene and that hits the optical system of the camera. In the end gives some variation. If you kept on taking pictures off the same scene with the camera, you will have small variations due to changes in the light. => **Variations in lighting**

- If you take pictures out with a telescope up into space, the atmosphere also causes some noise. Cloud that on the way could be the noise. => **Atmospheric effects** so that can change the amount of light that hits camera.

- Digital CCD chip is a piece of electronics and it's sensitive to things like **temperature**, **any electrical noise** that might come from the power source. => Cause some random fluctuations of the amount of light that is measured by the sensor.

- Sensor non-uniformity, dust, vibrations, lens distortion, focus limitations, sensor saturation (too much light), underexposure (too little light).

**Sampling**

Sampling resolution and quantization leads to aliasing effects.

**Processing**

When we do some processing in the camera collecting the light, some post processing after the sensor that can cause some numerical problems with precision. Overflow and approximation.

The camera automatically performs JPEG compression and in this way, we approximating the light and this is sort of a random fluctuation that changes the output light distribution that we are observed with our camera.

**Image-encoding**: Artifacts from lossy image compression.

A reasonable model is to assume that imaging noise is independent of position and identically distributed (i.i.d).

##### Salt-n-pepper and additive Gaussian noise

This just shows you two examples of noise.

The top one is called salt and pepper or impulse noise. => Model of a sensor element failure (off or oversaturated). CCD chip is either permanently broke or just temporarily broken.

The **Gaussian nose** is the model of **accumulated system noise**. => We draw a random number from a Gaussian distribution that has zero mean and some variants and then we add that to the original value.

<div align=center><img src="https://i.imgur.com/OseYqxc.png" alt="image-20230213020554474" style="zoom: 25%" /></div>

Maybe we have a real camera and takes the original image. Besides the blurring effect of the camera, we also add some noise and map it to the imaging plane. These are just two examples of models of the type of noise that could occur in the process.

##### The effect of our model

It takes a point light source and then, it smears it out so that it covers several pixels not just one pixel.

**Simulation example**

<div align=center><img src="https://i.imgur.com/ZWJn3bP.png" alt="image-20230213022318018" style="zoom:25%;" /></div>

The smearing effect comes from the $PSF$, and the noise is using the i.i.d $Gaussian$ noise.

### Modeling the Point Spread Function (PSF)

Now we need to develop a model for $PSF$ which is based on the assumption of a lienar operator.

#### Linear superposition integral

We are still aiming at modeling the imaging system.

$PSF$ is that it should model the light gets smeared out. <u>A point light source that doesn't end up in a point in image plane but gets smeared out in the image plane.</u> That's what we want the $PSF$ does.

In the first step, we could assume that the $PSF$ operator is <u>linear superposition integral</u>. Lets model the $PSF$ operator as a position dependent weigthed average of the input - a superposition of the input.

We assume that we have some input distribution, for simplicity, we draw it in 2D but all the other things are the same. Leads to the same result.

If I consider a specific location in the imaging system, you could think of it as a specific pixel. How does that relate to light out in the real world out on his light distribution plane? What we assume is that there is some function $h$ that acts as a weighting function (something that has to do with $PSF$) that is multiplied together with the light distribution function $f$.

$$
g(x, y) = \int\int f(x', y')h(x, y; x', y') dx'dy'
$$

<div align=center><img src="https://i.imgur.com/dqvdcsD.png" alt="image-20230213025949187" style="zoom:25%;" /></div>

The double integral equation here is saying that, to figure out what is the light distribution at $x, y$, we need to go via $h$ function, look at a specific location out on the input plane, and we need to consider all possible $(x', y')$ => all possible locations on the input plane. Let the contribution be weighted by $h$ function.

<div style="font-family: 'Noto Serif SC'">意思是对于输出层的某一个像素点，他有可能是输入层所有像素点均等权重相加的结果，也有可能是分配权重不均衡导致某几个权重大的点的影响因素较多的结果。</div>

**<u>It's weird because it seems like we have an function that depnds on both the coordinate system out on the real world and the coordinate system on the image plane.</u>**

The way to think of this is that we allow $h$ to be different to change as we look at different places in the output. So you could maybe imagine a camera that doesn't behave similarly for all the pixels in the image plane. => **Position Dependent**

But for ordinary cameras, this is not needed.

#### Linear shift-invariant systems - the convolution integral

Usually an imaging system has a $PSF$ which is **<u>independent of position</u>**, except telescope.

But if we assume that the $PSF$ is position independent, then from the superposition integral (equation before), we can simplify the integral. That means we can change $h$.

So what we will assume is that $h$ does not directly depend on $x$ and $y$. There exists some $h$ function of two variables what we need is that we shift this $h$ function around to simulate the model $PSF$.


$$
h(x, y;x',y') = h(x-x', y-y')
$$


This simplifies the superposition integral to 


$$
g(x, y) = \int^{\infin}_{-\infin}\int^{\infin}_{-\infin}f(x', y')h(x-x',y-y')dx'dy'
$$


This is called the **<u>convolution integral</u>**.

You will also see different notations for this because it's pretty difficult annoying to keep having to write out double integrals.

Instead of writing double integral, we use 

**2D**


$$
g(x, y) = \{f*h\}(x,y)
$$

**1D**

$$
g(x) = \{ f*h \}(x)
$$

*****

$f*h \Leftrightarrow$ Apply convolution integral on the two function $f$ and $h$.

---

<div align=center><img src="https://i.imgur.com/bq11zFg.png" alt="image-20230213034935897" style="zoom:25%;" /></div>

## Signal processing basics

### Dirac delta (impulse) functions

Haven't been taught right now.

### What causes the PSF?

**Finite apertures (pinhole) and sensors.**

<div align=center><img src="https://i.imgur.com/r2FnSy9.png" alt="image-20230213040551883" style="zoom:25%;" /></div>

Let's consider a simplified imaging system where the camera is 1D or it's a line scanner. The input signal $f(x')$ is 1D.

So $x'$ is the input space. We will consider the perfect camera case. This is the case where the aperture the pinhole is infinitely small. It's through a point or sensor, the centering element in our chip, this is also infinitely small. In this case, we have this idealized setup because here the $PSF$ and $h$ function for the $PSF$ becomes a $\delta$ function. In this case, we get an exact copy of the input. 

<div align=center><img src="https://i.imgur.com/14i1N66.png" alt="image-20230213121226260" style="zoom:25%;" /></div>

We model the effect of the apeture with $h(x)$. And this function is part of the $PSF$ of the camera. So we only need to worry about the aperture and this is the cause of the $PSF$.

<div align=center><img src="https://i.imgur.com/hKUk0Pp.png" alt="image-20230213121533071" style="zoom: 25%"/></div>

$h_d(x)$ and $h(x')$ together form the final output image and they form the system $PSF$. So, how does this work?

You can think of it as what is the amount of light that goes throuh the finite aperture? We know that's given by convolving with the corresponding $h$ function and the input signal $f$. And this gives you some intermediate result just after the finite aperture. => $g(x)$.

Now we have this finite sensor as well and that we take whatever comes out of the aperture, and we convolve that by $h_d$. And that is the resulting image given by our camera.

We can definitely write our resulting image as below.


$$
I(x) = f(x')*h(x)*h_d(x)
$$


But we can combine these together and find a combined $h$ function that both models the finite aperture and the finite sensor size. We cam do this simply by convolving the two $h$ together prior to convolving wiht the input signal and in practice, it's pretty difficult to separate the different effects from each other.

On top of this, we might also see blurring effects from other parts of the imaging system, e.g., from optical lenses, which will add yet another term to the $PSF$.

### Example of simulating low photon count

<div align=center><img src="https://i.imgur.com/IhDiZPq.png" alt="image-20230213122628472" style="zoom: 25%"/></div>

Some times, we can get a more visible version compared to the original picture.
