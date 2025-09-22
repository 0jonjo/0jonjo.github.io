---
layout: post
title: "Immersing in AI: Notes from a Developer Studying Machine Learning, Ethics and Data"
description: "Parallels between an AI course and work as a developer on AI projects"
tags: programming learning
categories: misc
youtubeId:
image:
---

This semester, I chose Supervised Machine Learning and Ethics and Data as my first courses in the [Artificial Intelligence program at UFRN](https://pes.imd.ufrn.br/pes/index). The first deals with algorithms that learn from examples, covering everything from data collection, systematization, and adjustments to the construction of decision trees and artificial neural networks. The goal is that, by the end of the semester, we'll be able to build predictive models from real datasets. For example, a machine could receive a patient's exam data and predict whether or not they have a predisposition to a certain disease. The second course, on the other hand, covers the historical, philosophical, and legal dimensions of using data and algorithms, including topics like bias, privacy, transparency, and responsibility.

In these first few months of the program, I’ve drawn several connections to the backend work I’ve been doing over the last few years, especially with the AI projects I'm working on at [JetRockets](https://www.linkedin.com/company/jetrockets/). Some of these connections relate to mathematical questions; others, to methods for adjusting data sent to and received from AI models; and some deal directly with ethics and privacy.

Here are the main connections I've observed:

- **Communicating with external systems and models**: A common aspect of large backend projects is integration with various third-party systems via queues, APIs, and webhooks. Beyond the purely technical details, this communication requires developing abstractions that can produce and consume information designed for other business rules—for example, payment gateways, ERP systems, and CRMs. The combination, smoothing, and transformation of data from different sources is a constant challenge.

In Machine Learning, during the training that gives rise to models, we study and practice how to interpret and prepare data from various sources for use in training and testing. It's necessary to understand the format, structure, and meaning of the data, as well as to identify and handle potential problems like missing values, inconsistencies, and noise.

Something similar occurs when communicating with AI models via API (like Claude, OpenAI, and Gemini): you need to interpret and prepare the input and output data to ensure it's in the correct format and meets the model's requirements. In both cases, a thorough understanding of the data and system needs is essential for effective communication.

- **Understanding vectorization**: In AI projects, vectorization is the process of transforming data into numerical vectors so it can be processed by machine learning algorithms. This is especially important when working with unstructured data like text and images. Vectorization allows algorithms to extract relevant features and facilitates analysis and decision-making.

In backend projects, vectorization can also optimize data storage and querying, particularly when working with large volumes of information. Understanding its concepts and techniques has been fundamental in the projects I'm involved in; studying and practicing them has been an important part of my AI learning journey.

- **Normalizing data**: Various data merging, comparison, and transformation processes involve normalization—adjusting values to a common scale without distorting differences in value ranges. This ensures data is handled consistently, especially in scale-sensitive algorithms.

In Machine Learning, normalization is a crucial data preprocessing step that improves the performance and convergence of algorithms. In AI projects, steps like preparing data for RAG (Retrieval-Augmented Generation) also involve normalization to meet model requirements. For this reason, it has been useful to revisit concepts like mean, median, mode, variance, and standard deviation, applying them in both my coursework and the AI projects I'm developing.

### Understanding and Controlling Bias in Data and Algorithms

Bias refers to the political, social, or gender inclinations present in data sources. They can have historical and cultural roots or be a product of conscious or unconscious human decisions. They influence everything from the improper selection of data to the unequal representation of different groups.

In Machine Learning, controlling bias is a constant concern, as it can lead to unfair or inaccurate results, especially when working with sensitive data. Bias control also appears in the preparation of prompts for AI models, where it's necessary to ensure the model's response is appropriate for the query, often by mitigating undesirable biases. I've been studying techniques and strategies to identify, mitigate, and control biases, ensuring fairer, more transparent, and responsible models.

### Managing Private Data

When dealing with personal and sensitive information, ensuring data privacy and security is essential. This involves adopting practices and technologies to protect against unauthorized access, leaks, and misuse.

In the relationship with AI models, managing private data is a constant concern. It is also critical to ensure compliance with privacy laws and regulations, such as the LGPD in Brazil. I have been studying legislation and best practices, adopting techniques and strategies to protect data privacy with secure and responsible approaches.

![image](https://github.com/0jonjo/0jonjo.github.io/assets/64807181/9364065b-f8c8-489c-a2e3-55bff1acd8c1)
>Image: "Mergulho" de Anderson Freire. [Open Verse, Creative Commons 2.0](hhttps://openverse.org/image/b952887f-8c48-4bb7-a5b6-e9ea9f97a00d)
