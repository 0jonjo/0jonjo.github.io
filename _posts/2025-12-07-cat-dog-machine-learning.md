---
layout: post
title: "Cat vs. Dog: A Machine Learning Experiment"
description: "A hands‑on pipeline: feature engineering, dimensionality reduction, model tuning and statistical comparison — step‑by‑step notebooks linked"
tags: programming learning ai english
categories: misc
youtubeId:
image: https://github.com/user-attachments/assets/af3b25bd-a228-4ee5-84e9-08e87fe5ebfe
---

"Cat vs. Dog" is the "Hello World" of Computer Vision. But solving it is one thing; understanding the mathematical and statistical foundations behind the solution is another.

For my final project in IMD3002 - Supervised Machine Learning at [Artificial Intelligence program at UFRN](https://pes.imd.ufrn.br/pes/index), instructed by Prof. João Carlos Xavier Junior, I didn't just trained models. We build a complete, rigorous scientific framework to test the fundamentals of Machine Learning.

Here is how I broke down the workflow into a series of sequential experiments.

### The Objective

The goal was to classify images of specific breeds (Miniature Pinscher/English Setter vs. Birman/Ragdoll) by exercising every fundamental stage of a classic ML pipeline: from raw pixels to statistical validation.

### Step 1: Feature Extraction (Beyond Pixels)

Raw images are noisy and high-dimensional. To make them digestible for classical algorithms, I implemented two distinct visual descriptors:

- HOG (Histogram of Oriented Gradients): To capture the shape and edge structures.
- LBP (Local Binary Patterns): To capture the texture of the fur.

### Step 2: Dimensionality Reduction

With high-dimensional feature vectors, the "Curse of Dimensionality" becomes a real threat. I applied PCA (Principal Component Analysis) to project the data into a lower-dimensional space, balancing computational efficiency with information retention.

### Step 3: Model Training & Tuning

I implemented and compared five distinct classes of algorithms to see how they handled the visual data:

- k-NN: Instance-based learning.
- Naive Bayes: Probabilistic modeling.
- Decision Trees: Rule-based learning.
- MLP (Multi-Layer Perceptron): Neural Networks.
- Ensembles: Random Forest, AdaBoost, Voting, and Stacking.

Each model underwent rigorous hyperparameter tuning (GridSearch) to ensure we were comparing the best versions of each.

#### Step 4: Statistical Evaluation

This is where many projects stop, but scientific rigor requires more than just comparing average accuracy. I applied the Friedman Test to determine if the differences in performance were statistically significant and the Nemenyi Post-hoc Test to pinpoint exactly which models outperformed the others.

### The Verdict

The statistical analysis revealed that for this specific dataset, an MLP combined with LBP features offered the best trade-off between predictive power and computational cost, significantly outperforming shape-based approaches (HOG).

You can explore the full step-by-step workflow, organized in sequential Jupyter Notebooks, on my GitHub: [https://github.com/0jonjo/cat_dog_ml](https://github.com/0jonjo/cat_dog_ml)

<img width="551" height="1023" alt="image" src="https://github.com/user-attachments/assets/af3b25bd-a228-4ee5-84e9-08e87fe5ebfe" />

Image: "Ai is... Banner", Rick Payne and Team. [Better Images of AI, Creative Commons 4.0](https://betterimagesofai.org/images?artist=RickPayneandteam&title=Aiis...Banner)
