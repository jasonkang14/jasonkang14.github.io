---
title: "What Is Pruning in Deep Learning?"
date: "2024-10-061T20:35:37.121Z"
template: "post"
draft: false
slug: "/ai/pruning-in-deep-learning"
category: "AI"
tags:
  - "AI"

description: "Explore pruning in deep learning, a technique for reducing model size and improving efficiency"
---

Llama3.2 was released last month, featuring medium-sized vision LLMs (11B and 90B) alongside lightweight, text-only models (1B and 3B). Earlier this year, I worked on a mobile application designed to assist users in environments such as refineries and power plants—critical infrastructure where safety regulations and data protection are paramount. Given the often unstable internet connections at these sites, I believed it would be advantageous to explore one of the lightweight text-only models for both performance and security reasons. Consequently, I reviewed the [Llama3.2 release notes](https://ai.meta.com/blog/llama-3-2-connect-2024-vision-edge-mobile-devices/) to gain insights into its development.
<br><br>
# Vision models

- Llama3.2 models (11B and 90B parameters) are the first in the Llama series to support vision tasks.
- Requires a new architecture for image reasoning to integrate visual input with existing language capabilities.


## Adding Image Input Support

- Adapters use cross-attention layers to process image representations within the language model.
  - Adapters are small neural network modules inserted into the layers of a pre-trained model 
  - Their primary purpose is to introduce new capabitlies to the model, which is image processing in this case
    - Adapters are added between the layers of the pre-trained model
    - They learn task-specific knowldge and dapt the outputs of each layer to the new task, which is image processing 
  - In the context of Llama3.2, the adapters connect the image encoder to the language model using cross-attention mechanisms. 
- Introduces adapter weights to integrate a pre-trained image encoder into the language model.
  - Adapter weights refer to the learnable parameters within the adapter modules.
  - These weights are adjusted to align the new input modality with the existing model while maintaining the core model
  
## Training Pipeline
### Pre-Training:
  - Starts with pre-trained Llama3.1 text models.
  - Image adapters and encoders are added.
  - Pre-training uses a large-scale dataset of noisy image-text pairs.
    - `Noisy` image-text pairs refer to datasets where the association between images and their corresponding textual descriptions is imperfect
    - It is not that the images are labeled with wrong descriptions, its more like descriptions with less detail, and the model is supposed to find the detailed descriptions through training 

### During training:
  - Image encoder parameters are updated.
  - Language model parameters remain unchanged, preserving Llama3.1's original text-only capabilities.

### Fine-Tuning:
  - Uses a medium-scale dataset of high-quality, in-domain, and knowledge-enhanced image-text pairs.
  - to refine the model's capabilities for specific tasks and improve its overall performance

### Post-Training and Alignment
- **Supervised Fine-Tuning**: Conducts rounds of fine-tuning using supervised learning.
- **Rejection Sampling**: Applies sampling methods to select the best-performing outputs.
- **Preference Optimization**: Directly optimizes model outputs for better performance.


## Synthetic Data Generation
- Employs Llama3.1 to filter and augment questions and answers based on in-domain images.
  - According to the [Llama3.1 Release Note](https://ai.meta.com/blog/meta-llama-3-1/), Llama3.1 405B is supposed to be good at synthetic data generation
- Uses a reward model to rank candidate answers, creating high-quality fine-tuning data.


## Safety and Agentic Capabilities
- Includes safety mitigation data to maintain the model's helpfulness and ensure safe operation.
- Enables deeper understanding and reasoning with multimodal (image and text) inputs.
- Advances Llama models toward more sophisticated agentic capabilities.


<br><br>

# Lightweight models

- Llama 3.2 introduces the first highly capable lightweight models (1B and 3B parameters) that can efficiently run on-device.
- These models are created using two key methods: pruning and distillation.


## Pruning:

- Used structured pruning on the Llama 3.1 8B model to create smaller, efficient versions (1B and 3B).
  - Pruning is a technique used to reduce the size of a neural network by removing less important or redundant parameters (weights, neurons, or filters) while attempting to maintain the model's performance.
  - The goal of pruning is to make the model more efficient, reducing its memory footprint, computational requirements, and potentially speeding up inference times
  - When a model is trained, not all the neurons or parameters contribute equally to its final performance. Pruning identifies and removes those less critical components, leading to a smaller and more optimized model.
- Involves systematically removing parts of the network while adjusting the remaining weights and gradients.
- Aims to reduce model size while retaining as much of the original model's knowledge and performance as possible.

## Knowledge Distillation:
- Uses a larger model (Llama 3.1 8B and 70B) as a "teacher" to transfer knowledge to the smaller 1B and 3B models.
- Incorporated logits from the larger models during the pre-training stage, using these outputs as token-level targets.
- Applied after pruning to help the smaller models recover performance.

## Post-Training Alignment:
- Uses a similar post-training process as Llama 3.1, involving multiple rounds of:
  - **Supervised Fine-Tuning (SFT)**: Further refining model behavior.
  - **Rejection Sampling (RS)**: Selecting the best-performing model outputs.
  - **Direct Preference Optimization (DPO)**: Directly optimizing the model based on preferred outputs.

- Scales context length support to 128K tokens while maintaining quality.
- Uses synthetic data generation, blending high-quality data for various capabilities like summarization, rewriting, instruction following, language reasoning, and tool use.

## Collaboration and Community Support:

- Worked with Qualcomm, Mediatek, and Arm to optimize models for mobile devices.
- Released model weights based on BFloat16 numerics.
- Actively exploring quantized variants for faster performance.