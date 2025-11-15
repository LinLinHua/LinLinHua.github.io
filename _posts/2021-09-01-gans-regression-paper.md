---
layout: post # Do not change
category: [ml] # Project categories
title: "Co-authored: Multi-Output Regression with GANs (MOR-GANs)" # Title from paper
date:   2021-09-01 10:00:00 +0100 # Date from resume
author: LilyHua # Your author nick
nextPart: _posts/2025-11-13-intent-classification-lstm.md
# Links to your next project
# prevPart: (This is your first project post, so no prevPart)
og_description: "Co-authored paper on using Wasserstein GANs (WGANs) as a novel multi-output regression method. Contributed to the software implementation in Python, Keras, and Tensorflow."
---

As a Research Assistant with the INHALE team at Imperial College London, I was a co-author on the paper **"Multi-Output Regression with Generative Adversarial Networks (MOR-GANs)"**, which was published in *Neural Computing and Applications*.

This research explored a novel application for Generative Adversarial Networks. While GANs are famous for image generation, they are rarely applied to non-image, small-dimensional data. Our work proposed and demonstrated a new method, **MOR-GAN**, which uses a **Wasserstein GAN (WGAN)** as a flexible, powerful regression method.

### The Challenge: Beyond Standard Regression

Traditional regression models, like Gaussian Process Regression (GPR), are powerful but often struggle with complex, real-world data distributions. They are typically designed for single-valued outputs and can fail when dealing with **multi-modal** data—where a single input can correspond to multiple valid output values.

A prime example of this challenge is data distributed in a non-linear, multi-modal fashion, such as an annulus (a ring or circle). A standard GPR model will fail, averaging the top and bottom of the circle and predicting a point in the empty middle.

### Our Innovation: The MOR-GAN Architecture

Our paper proposed leveraging the inherent power of WGANs to "learn" the entire underlying data distribution, rather than simply mapping inputs to single-point outputs. This capability allows MOR-GAN to effectively model and predict from complex, multi-modal distributions.

Below is the detailed WGAN structure we implemented, consisting of a Generator and a Critic (Discriminator) working in tandem to refine predictions.

<div class="sx-center">
    <div class="sx-picture">
    <a href="/assets/img/posts/wgan_structure.png" data-lity>
        <img src="/assets/img/posts/wgan_structure.png" alt="WGAN Architecture Diagram"/>
    </a>
    <span class="sx-subtitle">Figure 1 (adapted): The WGAN (Generator and Critic) structure implemented for the MOR-GANs model.</span>
    </div>
</div>

### My Contribution: Software Implementation & Analysis

As formally stated in the "Author Contributions" section of the paper, my key role was in the **software development**.

I was responsible for implementing and testing the various deep learning models using **Python**, **TensorFlow**, and the **Keras** API. A significant part of my work involved conducting a rigorous comparative analysis of the WGAN model's performance against several other established deep learning architectures, including:

* **AutoEncoders (AEs)**

* **Recurrent Neural Networks (RNNs)**

* **Convolutional Neural Networks (CNNs)**

### Compelling Results: Outperforming Traditional Methods

The MOR-GAN's ability to model complex, multi-modal distributions proved highly effective. While traditional GPR models would fail on the "Annulus" (circle) dataset, our MOR-GAN model successfully learned to capture the intricate, multi-modal nature of the ring.

The plots below show the generated data (red dots) learning to match the true data distribution (blue dots), demonstrating that the model is not "averaging" the results but is correctly identifying the valid, multi-modal output space.

<div class="sx-center">
    <div class="sx-picture">
    <a href="/assets/img/posts/wgan_distribution.png" data-lity>
        <img src="/assets/img/posts/wgan_distribution.png" alt="WGAN Distribution Results on Annulus Dataset"/>
    </a>
    <span class="sx-subtitle">Figure (adapted): WGAN successfully models the multi-modal annulus, with generated (red) and true (blue) data.</span>
    </div>
</div>

This research definitively demonstrated that GANs are a highly viable and flexible tool for complex regression tasks, capable of significantly outperforming traditional methods.

---

### Project Links & Publication

{% assign publication_url = 'https://www.mdpi.com/2076-3417/12/18/9209' %}

<div class='sx-button'>
  <a href='{{ publication_url }}' class='sx-button__content red' target='_blank' rel='noopener noreferrer'>
    <img src='{{ site.baseurl }}/assets/img/icons/info.svg' alt="Publication Icon"/> View Publication
  </a>
</div>