---
title: "Dissecting Llama3.2"
date: "2024-10-27gT20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/dissecting-llama-32"
category: "LLM"
tags:
  - "LLM"

description: "A shot at understanding Llama3.2"
---

Llama3.2 was released last month, featuring medium-sized vision LLMs (11B and 90B) alongside lightweight, text-only models (1B and 3B). Earlier this year, I worked on a mobile application designed to assist users in environments such as refineries and power plants—critical infrastructure where safety regulations and data protection are paramount. Given the often unstable internet connections at these sites, I believed it would be advantageous to explore one of the lightweight text-only models for both performance and security reasons. Consequently, I reviewed the [Llama3.2 release notes](https://ai.meta.com/blog/llama-3-2-connect-2024-vision-edge-mobile-devices/) to gain insights into its development.
<br><br>
# Vision models

- Llama3.2 models (11B and 90B parameters) are the first in the Llama series to support vision tasks.
- Requires a new architecture for image reasoning to integrate visual input with existing language capabilities.


## Adding Image Input Support

- Adapters use cross-attention layers to process image representations within the language model.
  - Adapters are small neural network modules inserted into the layers of a pre-trained model 
  - Their primary purpose is to introduce new capabilities to the model, which is image processing in this case
    - Adapters are added between the layers of the pre-trained model
    - They learn task-specific knowledge and adapt the outputs of each layer to the new task, which is image processing 
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
  - Instead of using the `noisy` image-text pairs used in pre-training, medium-scale high quality in-domain and knowledge-enhanced image-text pair data is used
    - This approach helps refine the model's understanding and performance, similar to how knowledge distillation leverages refined outputs to improve model training.

### Fine-Tuning:
  - Uses a medium-scale dataset of high-quality, in-domain, and knowledge-enhanced image-text pairs.
  - to refine the model's capabilities for specific tasks and improve its overall performance

### Post-Training and Alignment
- **Supervised Fine-Tuning**: Conducts rounds of fine-tuning using supervised learning.
- **[Rejection Sampling](https://arxiv.org/abs/2309.06657)**: Applies sampling methods to select the best-performing outputs.
  - a technique used to filter or modify model outputs by setting specific criteria that the outputs must meet
  - used when the outputs may not align with desired constraints or quality standards 
    - I believe this was crucial as the model size is really small. Meta must have set some sort of standards for the smaller models to meet before releasing them
  - this also helps with safety measures
  
- **[Direct Preference Optimization(DPO)](https://arxiv.org/abs/2305.18290)**: Directly optimizes model outputs for better performance.
  - this goes well with the rejection sampling, as the purpose of DPO is to align the outputs of the model with the standards that had been previously set
  - uses preference data to adjust the model, reducing potential noise introduced by the reward model.
  - this will be the topic of the next post

## Synthetic Data Generation
- Employs Llama3.1 to filter and augment questions and answers based on in-domain images.
  - According to the [Llama3.1 Release Note](https://ai.meta.com/blog/meta-llama-3-1/), Llama3.1 405B is supposed to be good at synthetic data generation
- Uses a reward model to rank candidate answers, creating high-quality fine-tuning data.
  - this aligns with the principle of Direct Preference Optimization(DPO)


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
- I have written about pruning and knowledge distillation in this [post](https://jasonkang14.github.io/ai/pruning-and-knowledge-distillation)


# Post-Training
- Scales context length support to 128K tokens while maintaining quality.
- Uses synthetic data generation, blending high-quality data for various capabilities like summarization, rewriting, instruction following, language reasoning, and tool use.

The detailed examination of the release notes has heightened my interest in experimenting with Llama3.2. Meta's emphasis on its enhanced capability to interpret charts and graphs suggests potential improvements in building a more robust Retrieval Augmented Generation (RAG) pipeline, particularly for processing PDF documents. In my upcoming analysis, I plan to explore Direct Preference Optimization (DPO) further and concurrently evaluate Llama3.2's efficacy in data processing.