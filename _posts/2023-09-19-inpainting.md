---
layout: post
title:  Inpainting
date: 2023-09-19 08:00:00
description: Learning note for inpainting.
tags: ssl
categories: study ucph
related_posts: true
toc: 
  sidebar: left
giscus_comments: false
thumbnail: assets/img/inpainting.jpg
---

Inpainting, as known as **predict missing pixels**.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
       <img src="https://i.imgur.com/KLK2ZIN.png" alt="image-20230919201623320" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 1. Context Encoders: Feature Learning by Inpainting (Pathak et al., 2016)
</div>

***Disclaimer: Below are the extraction from the original paper.*** It will be reposted once I finished read all of them when I am available.

***Reference: Context Encoders: Feature Learning by Inpainting***

---

#### Abstraction

We present **<u>an unsupervised visual feature learning algorithm</u>** driven by context-based pixel prediction.

Context Encoders - a **convolutional neural network** trained to **generate the contents of an arbitrary image region conditioned on its surroundings**.

To succeed at this task, context encoders need to both **understand the content of the entire image,** as well as **produce a plausible hypothesis for the missing parts**.

When training context encoders, we have experimented with both **a standard pixel-wise reconstruction loss**, as well as **a reconstruction plus an adversarial loss**.

The **latter produces much sharper results** because it can **better handle multiple modes** in the output.

A context encoder learns **a representation that captures not just appearance** but also the **semantics of visual structures**. ==> *Question*

Context encoders can be used for **semantic inpainting tasks**, either **stand-alone** or **as initialization for non-parametric methods**.

#### Introduction

Our visual world is very diverse, yet **highly structured**, and humans have an uncanny ability to make sense of this structure.

This work is to explore **whether state-of-the-art computer vision algorithms can do the sam**e.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
       <img src="https://i.imgur.com/nFaEBX3.png" alt="image-20230919205428308" class="img-fluid rounded z-depth-1" data-zoomable />
    </div>
</div>
<div class="caption">
  Figure 2. Demonstration
</div>

Although the center part of the image is missing, most of us can easily imaging its content from the surrounding pixels, without having ever seen that exact scene. Some of us can even draw it.

This ability comes from the fact that **natural images, despite their diversity, are high structured**. In this case, the regular pattern of windows on the facade. We **humans** are able to **understand this structure** and **make visual predictions** even when seeing **only parts of the scene**.

In this paper, we show that **it is possible to learn and predict this structure** using **convolutional neural networks** (CNNs), a class of models that have recently shown success across a variety of **image understanding tasks**.

Given **an image with a missing region**, we train a convolutional neural network to **regress to the missing pixel values**. *Question*

We call our model **context encoder**, as it consists of **an encoder capturing the context of an image into a compact latent feature representation** and **a decoder which uses that representation to produce the missing image content**.

The context encoder is closely related to auto encoders, as it shares a **similar encoder-decoder architecture**.

**Autoencoders** **take an input image** and try to **reconstruct it** after it passes through a low-dimensional "bottleneck" layer, with the aim of **obtaining a compact feature representation** is likely to **just compresses the image content without learning a semantically meaningful representation**. ==> Encoder has done the abovementioned, the decoder is to map back to the original datashape.

---

ChatGPT gives the following explaination to autoencoder.

1. "**Autoencoders**": Autoencoders are a class of neural network architectures used for unsupervised learning. They consist of an encoder and a decoder, and their <u>primary purpose is to learn a compressed representation of the input data</u>.
2. "**Take an input image**": Autoencoders typically operate on input data, which in this case is an image. The image is passed through the network.
3. "**Reconstruct it**": The <u>core task of an autoencoder is to take the input image</u> and attempt to **<u>generate an output (reconstruction) that is as close as possible to the original input</u>**. **<u>In essence, it tries to replicate the input</u>**.
4. "**After it passes through a low-dimensional 'bottleneck' layer**": Autoencoders have an intermediate layer called the bottleneck or latent space, which has a lower dimension than the input data. This layer forces the network to **<u>learn a compressed representation of the input</u>** data.
5. "**With the aim of obtaining a compact feature representation**": The primary objective of this compression process is to **<u>obtain a compact feature representation of the input</u>** data. This means that the autoencoder is trying to <u>capture the most important and salient features of the input</u> data in a condensed form.
6. "**Is likely to just compresses the image content without learning a semantically meaningful representation**": This part of the statement suggests <u>a potential limitation of traditional autoencoders</u>. In some cases, when using basic autoencoder architectures and simple loss functions like mean squared error, the learned compressed representation may indeed **<u>focus more on compressing the raw image content</u>** (e.g., pixel values) rather than **<u>capturing higher-level, semantically meaningful features</u>** (e.g., object shapes or meaningful abstractions).

In other words, **traditional autoencoders** may prioritize achieving a low reconstruction error (**<u>faithfully reproducing the input</u>**) over learning abstract, semantic information. However, this limitation can be addressed or mitigated by modifying the architecture, loss functions, or training strategies, as mentioned in the previous response. **<u>More advanced autoencoder</u>** variants, like variational autoencoders (VAEs) or denoising autoencoders, **<u>are designed to encourage the learning of semantically meaningful representations</u>**.

---

**Denoising autoencoders** address this issue by **corrupting the input image** and requirng the network to **undo** the damage. However, this corruption process is typically very **localized and low-level**, and does **not require much semantic information** to undo.

---

ChatGPT explains what are localized and low-level. :point_down:

1. "**Localized**": When it is said that the corruption process is "localized," it means that the noise or distortion introduced to the input data **tends to affect only a small portion of the data**. In the context of images, for example, localized corruption might involve adding noise to individual pixels or small regions of the image. It does not involve large-scale changes or alterations to the entire image.
2. "**Low-level**": The term "low-level" refers to the nature of the corruption. Low-level corruptions are simple and typically involve basic operations on the data, such as adding random noise, blurring, or altering individual data points (e.g., pixel values in an image). These corruptions do not carry complex semantic meanings or involve high-level abstractions. They are often applied at the level of raw data elements.

In the context of denoising autoencoders, the idea is that **the noise or distortions introduced to the input** data are **relatively simple** and **affect only small parts** of the data. Because these **corruptions are not complex or semantically rich**, the network may **not need to learn deep**, **high-level semantics to undo** the effects of the corruption. Instead, it can focus on relatively shallow and low-level operations to recover the clean data. This is why the statement suggests that denoising autoencoders may not necessarily learn highly meaningful semantic representations since the corruption process itself is not very demanding in terms of semantic understanding.

---

In contrast, our context encoder needs to solve **<u>a much harder task</u>**: **<u>to fill in large missing areas of the image, where it can't get "hints" from nearby pixels.</u>** 

This requires **a much deeper semantic understanding of the scene**, and the **ability to synthesize high-level features over large spatial extents**.

For example, an entire window needs to be **conjured up "out of thin air"** in Figure 2 above. 

This is similar in spirit to wrd2vec which **<u>learns word representation from natural language sentences by predicting a word given its context</u>**.

Like autoencoders, **<u>context encoders</u>** are **<u>trained in a completely unsupervised manner</u>**.

Our results demonstrate that in order to succeed at this task, a model needs to **<u>both understand the content of an image</u>**, as well as **<u>produce a plausible hypothesis for the missing parts</u>**.

This task, however, is inherently **multi-modal** as there are **multiple ways** to **fill the missing region while also maintaining coherence with the given context**.

We **decouple this burden** in our loss function by **jointly training our context encoders** to **minimize both a reconstruction loss and adversarial loss**.

The reconstruction loss (L2) **captures the overall structure** of the missing region in relation to the context, while the adversarial loss has the effect of **picking a particular mode** from the distribution.

- "**Reconstruction loss (L2)**": This loss function measures how well the generated output (the completed or inpainted region) matches the original data. It's often calculated as the mean squared difference between the generated output and the ground truth (original data). In other words, it captures how well the generated content resembles the actual missing region in terms of overall structure.
- "**Adversarial loss**": This type of loss function is typically associated with GANs. It involves a discriminator network that tries to distinguish between real (ground truth) data and generated data. The adversarial loss **encourages the generated data to be more realistic** and closer to the distribution of real data. In this context, it is mentioned that the adversarial loss has the effect of selecting a particular mode from the distribution, which means **it encourages the generated content to be more similar to specific instances of real data** (modes) rather than just any possible output.

So, the overall strategy described here is to **simultaneously train context encoders** to **generate missing regions** in a way that **not only captures the overall structure** (reconstruction loss) but also **encourages the generated content to resemble specific instances of real data** (adversarial loss). This approach aims to **strike a balance between maintaining coherence with the context and producing realistic**, data-driven details in the inpainted or completed regions.

Figure 2 shows that using only the **reconstruction loss produces blurry results**, whereas **adding the adversarial loss results in much sharper predictions**.

**<u>Evaluate the encoder and the decoder independently.</u>** 

On the encoder, we show that **encoding** just the **context of an image patch** and using the **resulting feature** to **retrieve nearest neighbor contexts from a dataset** produces patches which are semantically similar to the original (unseen) patch.

- After encoding the context of an image patch, the authors take the **resulting feature representation** and **use it to find other image patches in a dataset that have similar feature representations**. In other words, they **look for other patches that share similar contextual characteristics** based on the extracted features.
- The key result or observation here is that when they retrieve image patches from the dataset based on the feature representation of the context, the **retrieved patches are found to be "semantically similar" to the original, unseen patch**. This means that **the retrieved patches share meaningful visual or structural characteristics with the original patch**, even though the encoder processed only the context of the patch.

**By encoding only the context of an image patch** and using the **resulting features to find similar contexts in a dataset**, they can **retrieve other patches that semantically similar to the original patch**. This suggests that the context encoder is **effective at capturing meaningful contextual information from images**, even without considering the patch itself.

We further validate the quality of the learned feature representation by fine-tuning the encoder for a variety of image understanding tasks, including classfication, object detection, and semantic segmentation.

We are competitive with the state-of-the-art unsupervised/self-supervised methods on those tasks. 

On the decoder side, we **show that our method is often able to fill in realistic image content**.

Indeed, to the best of our knowledge, ours is the **first parametric inpainting algorithm** that is able to **give resonable results for semantic hole-filling** (large missing regions).

The context encoder can also be useful as a better visual feature for computing nearest neighbors in non-parametric inpainting methods.

#### Related work

Computer vision has made tremendous progress on **semantic image understanding tasks** such as classification, object detection, and segmentation in the past decade.

Recently, **Convolutional Neural Networks (CNNs)** have greatly advanced the performance in these tasks. The success of such models on image classification paved the way to tackle harder problems, including unsupervised understanding and generation of natural images.

We briefly review the related work in each of the sub-fields pertaining to this paper.

##### **Unsupervised learning**

**CNNs trained for ImageNet classification** with over a million **labeled examples** learn features which generalize very well across tasks. However, **whether such semantically informative and generalizable features can be learned from raw images alone, without any labels**, remains an open question. Some of the earliest work in deep unsupervised learning are autoencoders. Along similar lines, denoising autoencoders reconstruct the image from local corruptions, to make encoding robust to such corruptions. While context encoders **could be thought of as a variant of denoising autoencoders**, the corruption applied to the model's input **is spatially much larger**, requiring **more semantic information** to undo.

##### Weakly-supervised and self-supervised learning

Very recently, there has been significant interesting in learning meaningful representations using weakly-supervised and self-supervised learning. One useful source of supervision is to **use the temporal information** contained in videos. **Consistency across temporal frames** has been used as supervision to learn embeddings which perform well on a number of tasks. Another way to use consistency is to track patches in frames of video containing task-relevant attributes and use the coherence of tracked patches to guide the training.

Most closely related to the present paper are efforts at **exploiting spatial context as a source of free and plentiful supervisory signal**. Recently, Doersch et al. used the task of **predicting the relative positions of neighboring patches within an image** as a way to train an unsupervised deep feature representations. 

Our context encoder solves **a pure prediction problem** (what pixel intensities should go in the hole?).

In contrast, **context encoders can be applied to any unlabeled image database** and **learn to generate images based on the surrounding context**.



The later part in the article is not that related to my interests. I will repost this blog as soon as possible to form my own interpretation focus on the technique, **inpainting**.
