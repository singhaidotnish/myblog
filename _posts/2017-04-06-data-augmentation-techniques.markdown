---
layout: post
title: Data Augmentation Techniques and Dataset resources 
date: 2025-02-22 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Shillong, Small, City]
---

What is Data Augmentation?
Data augmentation is a technique used to artificially expand the size and diversity of a dataset by applying various transformations to existing data. This helps improve the performance and generalization of machine learning models, especially when training data is limited.

Common Data Augmentation Techniques
**For Image Data**

>Geometric Transformations:
    Rotation
    Scaling
    Translation (shifting)
    Flipping (horizontal/vertical)
    Cropping

>Color and Intensity Transformations:
    Brightness adjustment
    Contrast enhancement
    Gamma correction
    Color jittering

>Noise Injection:
    Adding Gaussian noise
    Adding salt-and-pepper noise
    Blurring (Gaussian blur, motion blur)
    Style Transfer & Domain Adaptation:
    
    Applying artistic filters (e.g., GAN-based augmentations)
    Simulating real-world conditions (e.g., rain, fog, shadows)

>Mixing & Cutout Techniques:
    CutMix: Mix patches of two different images
    MixUp: Blend two images and labels
    Random Erasing: Remove random parts of an image

>Synthetic Data Generation:
    Using Generative Adversarial Networks (GANs)
    Using diffusion models

>For Text Data
    Word Replacement & Synonym Augmentation (e.g., using WordNet or embeddings)
    Sentence Paraphrasing (e.g., back-translation)
    Character-level Augmentations (typos, shuffling, OCR-based distortions)
    Adding Noise (inserting/removing random words)
    Text Mixing (cut-pasting sentences)

>For Time-Series Data
    Jittering (adding noise)
    Scaling & Normalization Variations
    Time Warping (stretching or compressing time intervals)
    Permutation (randomly shuffling parts of the sequence)
    Synthetic Time-Series Data Generation (GANs, RNN-based synthetic data)

>For Tabular Data
    SMOTE (Synthetic Minority Over-sampling Technique)
    Feature Combination & Splitting
    Data Masking & Missing Value Simulation
    Random Permutations

**Where to Find or Generate Datasets?**
**Public Datasets**
>**Image Datasets**
    ImageNet
    COCO
    OpenImages
    MNIST (Handwritten digits)

>**Text Datasets**
    Common Crawl
    Hugging Face Datasets
    Google’s NLTK Corpus

>**Time-Series Datasets**
    UCI Machine Learning Repository
    Yahoo Finance (for stock market data)
    PhysioNet (health monitoring data)

>**Tabular Datasets**
    Kaggle Datasets
    OpenML
    Google’s Dataset Search