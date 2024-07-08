---
layout: post
title: Graduation
date: 2024-06-28 10:15:00
featured: true
description: Congratulations to my graduation from the DIKU, University of Copenhagen.
tags: fun
categories: study ucph
related_posts: true
giscus_comments: true
thumbnail: assets/img/graduation-photo.jpg
---

I am thrilled to announce that I have successfully defended my master thesis and received my Master's Degree in Computer Science from the Department of Computer Science (DIKU), University of Copenhagen.

<div class="row mt-3 mb-3">
  <div class="col-sm mt-3 mt-md-0">
	<img src="{{ site.baseurl }}/assets/img/graduation-photo.jpg" alt="graduation-photo" class="img-fluid rounded z-depth-1" data-zoomable/>
  </div>
  <div class="col-sm mt-3 mt-md-0">
    <img src="https://i.imgur.com/7Ee4Raf.jpeg" alt="graduation-photo-2" class="img-fluid rounded z-depth-1" data-zoomable>
  </div>
</div>

The title of my thesis is "Exploration of Self-Supervised Learning Methods for Longitudinal Image Analysis", specifically focusing on the application of self-supervised learning methods, which are BYOL and SimSIAM, in medical image analysis for longitudinal studies (i.e., tumor progression) on lung cancer. The 3D ResNet model is first pre-trained on the LIDC-IDRI dataset, and then the model is tasked with predicting the tumor volume at the future timepoint of the same subject, on the Longitudinal 4D Lung dataset. The results show that the self-supervised learning methods cannot be used to train 3D ResNet to learn meaningful representations for longitudinal image analysis, and the learned representations are not significantly correlated with the actual tumor volume, verified by the linear probing task. This failure is attributed to the following reasons: 1. the volume featured by the LIDC-IDRI dataset (nodules) is with substatial differences from the Longitudinal 4D Lung dataset (tumors), 2. the current pre-processing pipeline does not isolate the lung region perfectly, 3. the model is not fine-tuned on the longitudinal downstream task, and 4. the invovled augmentations impair the model's ability to learn precise tumor volume representations.

<div class="row mt-3 mb-3">
    <div class="col-sm mt-3 mt-md-0">
		 <img src="https://i.imgur.com/16tSUKF.png" alt="2831691531627_.pic" class="img-fluid rounded z-depth-1" data-zoomable/>
    </div>
</div>

I am grateful for the support and guidance from my supervisor, friends, and family, along the way. Thank you to everyone who has been a part of this journey with me. 🎓🎉

Additionally, I am very excited to share the news that I am going to work as an AI engineer in a company, to embark on the next chapter of my life and look forward to new opportunities and challenges. 🚀
