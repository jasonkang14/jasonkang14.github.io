---
title: "How Is the Transformer Trained?"
date: "2024-12-01T20:35:37.121Z"
template: "post"
draft: true
slug: "/llm/how-is-the-transformer-trained"
category: "LLM"
tags:
  - "LLM"

description: "A detailed exploration of how the Transformer model, introduced in Attention Is All You Need, is trained." 
---

In Korea, there is a tradition of "study" groups where engineers from various companies or backgrounds gather to explore specific engineering concepts. Participants typically select a book or an online curriculum to follow, and each week, one member is responsible for studying a chapter in depth and presenting the ideas to the group. I recently joined a study group focused on understanding how Large Language Models (LLMs) function. Last week, I was tasked with presenting the concepts of Attention and Transformers within LLMs. While I am familiar with these concepts, explaining them to others proved to be a distinct challenge compared to understanding them on my own. Here is my perspective on explaining how the Transformer model in the paper [Attention Is All You Need](https://arxiv.org/abs/1706.03762) was trained.


# Understanding How the Transformer Model Was Trained


## What is the Transformer model

The Transformer model, introduced in the seminal paper **"Attention Is All You Need"**, is a neural network architecture specifically designed for sequence transduction tasks, such as machine translation. It represents a significant departure from prior approaches by eliminating recurrence and convolution entirely in favor of relying solely on attention mechanisms. 

## Model Architecture

Before delving into how the Transformer was trained in the paper, let's look at the overall architecture of the model. The **Transformer** model architecture is an encoder-decoder structure that replaces traditional recurrence or convolutional layers with **self-attention** and **feed-forward layers**. Here’s a detailed breakdown of its architecture:

### **1. Overview: Encoder-Decoder Structure**
The Transformer consists of:
- **Encoder**: Maps the input sequence into a continuous representation.
  - the encoder translates a natural language into tokens, and then turns these tokens into vectors in order for a computer to understand the human language
- **Decoder**: Uses the encoder's representation to generate the output sequence.
  - the decoder is the opposite of the encoder. It translates the vectors into tokens and then the tokens get translated into natural language in order for humans to understand

Both the encoder and decoder are composed of **stacked identical layers**, each containing attention mechanisms and feed-forward networks.

---

### **2. Encoder**
The encoder processes the input sequence into a sequence of continuous representations. There are four components in the encoder: **Multi-Head Self-Attention Layer**,  **Feed-Forward Network (FFN)**, **Residual Connections**, and **Layer Normalization**. Let's look at each component in more detail:

#### **1. Multi-Head Self-Attention Layer**

##### **What It Does:**
This layer computes attention for each token in the sequence with respect to every other token in the sequence, allowing the model to understand relationships and dependencies regardless of their positions.

##### **How It Works:**
- For a given token (e.g., "cat"), it looks at all other tokens in the sequence (e.g., "The" and "sleeps") and decides how much focus to give to each based on their importance for that token.
- Multiple attention "heads" allow the model to capture different types of relationships simultaneously (e.g., syntactic, semantic).

##### **Why It’s Important:**
This mechanism enables the encoder to:
- Identify relationships within the sequence, such as subject-verb agreement or modifying adjectives.
- Capture long-range dependencies, even if two tokens are far apart.

---

#### **2. Feed-Forward Network (FFN)**

##### **What It Does:**
The FFN is applied independently to each token's representation. It further processes and transforms the token embeddings to make them richer and more useful for downstream tasks.

##### **How It Works:**
- It is a simple two-layer neural network with a **ReLU activation**:
  \[
  FFN(x) = \text{ReLU}(xW_1 + b_1)W_2 + b_2
  \]
  - \( W_1 \) and \( W_2 \): Weight matrices for the two layers.
  - \( b_1 \) and \( b_2 \): Bias terms.
  - ReLU introduces non-linearity, allowing the network to model complex patterns.
- The FFN operates separately on each token's representation, treating them independently at this stage.

##### **Why It’s Important:**
The FFN adds capacity to the model to learn non-linear transformations, enhancing the ability of the encoder to represent complex features of the input data.

---

#### **3. Residual Connections**

##### **What It Does:**
Residual connections are shortcuts that add the input of a layer directly to its output.

##### **How It Works:**
- For a layer's output \( x_{\text{out}} \), a residual connection calculates:
  \[
  \text{Output} = \text{LayerNorm}(x + x_{\text{out}})
  \]
  - \( x \): The input to the layer.
  - \( x_{\text{out}} \): The processed output from the layer.

##### **Why It’s Important:**
- Helps gradients flow smoothly during training, mitigating the vanishing gradient problem.
- Allows the network to learn small incremental updates rather than completely new transformations at each layer.

---

#### **4. Layer Normalization**

##### **What It Does:**
Layer normalization stabilizes training by ensuring that the input to each layer has a consistent distribution of values.

##### **How It Works:**
- It normalizes the activations within a layer to have a mean of 0 and a variance of 1.
- For an input \( x \) to the layer:
  \[
  \text{Norm}(x) = \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}}
  \]
  - \( \mu \): Mean of the activations.
  - \( \sigma^2 \): Variance of the activations.
  - \( \epsilon \): A small value to prevent division by zero.

##### **Why It’s Important:**
- Reduces the internal covariate shift, making the model converge faster during training.
- Ensures that different layers operate on inputs with consistent scaling.

---

#### **Structure of Each Encoder Layer**
Each of the **N = 6 layers** in the encoder has the following structure:
1. **Multi-Head Self-Attention**:
   - Processes the input sequence and builds contextual representations.
2. **Residual Connection + Layer Normalization**:
   - Stabilizes training and aids learning.
3. **Feed-Forward Network (FFN)**:
   - Adds additional processing capacity.
4. **Residual Connection + Layer Normalization**:
   - Ensures smooth gradient flow and numerical stability.

This structure is repeated for all layers in the encoder, progressively refining the input sequence representations into context-rich embeddings that are passed to the decoder.

---

#### **Why These Components Work Together**
- **Multi-head self-attention** extracts dependencies across the sequence.
- **Feed-forward networks** enhance each token's representation individually.
- **Residual connections and layer normalization** ensure efficient and stable training.
This design makes the encoder highly effective at capturing the relationships and context of the input sequence.


---

### **3. Decoder**
The decoder generates the output sequence auto-regressively, one token at a time.

#### **Components:**
- **Masked Multi-Head Self-Attention Layer**:
  - Similar to the encoder’s self-attention but prevents positions from attending to future tokens to maintain causality.
- **Encoder-Decoder Attention Layer**:
  - Attends to the encoder's outputs to incorporate information from the input sequence.
- **Feed-Forward Network**:
  - Same as in the encoder.
- **Residual Connections** and **Layer Normalization**:
  - Also applied after each sub-layer.

#### **Structure**:
The decoder has **N = 6 layers**, with each layer consisting of:
1. Masked multi-head self-attention.
2. Encoder-decoder attention.
3. Point-wise feed-forward network.

---

### **4. Multi-Head Attention**
Multi-head attention is central to the Transformer's ability to capture complex relationships.

#### **Steps:**
1. **Linear Projections**:
   - Inputs are projected into **query (Q)**, **key (K)**, and **value (V)** matrices.
2. **Attention Scores**:
   - Compute attention scores using scaled dot-product attention:
     \[
     \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{Q K^T}{\sqrt{d_k}}\right)V
     \]
   - The scaling factor (\(\sqrt{d_k}\)) prevents overly large values when the dimensionality \(d_k\) is high.
3. **Multiple Heads**:
   - Perform attention independently for multiple "heads" to capture different aspects of the input.
4. **Concatenation**:
   - Combine the outputs of all heads and project them back into the original dimension.

---

### **5. Feed-Forward Networks (FFN)**
Each layer contains a fully connected feed-forward network applied position-wise.
- Structure:
  - First layer expands the dimensionality.
  - Second layer projects back to the original dimension.
  - Activation: ReLU.

---

### **6. Positional Encoding**
Since the Transformer does not use recurrence or convolution, **positional encodings** are added to the input embeddings to provide the model with information about the order of elements in the sequence.

#### **Formula:**
The positional encoding uses sinusoidal functions:
\[
PE(pos, 2i) = \sin\left(\frac{pos}{10000^{\frac{2i}{d_{\text{model}}}}}\right), \quad
PE(pos, 2i+1) = \cos\left(\frac{pos}{10000^{\frac{2i}{d_{\text{model}}}}}\right)
\]

This ensures that the encodings are unique for each position and that they generalize well to unseen sequence lengths.

---

### **7. Model Hyperparameters**
- **Input/Output Dimension (\(d_{\text{model}}\))**: 512
- **Number of Attention Heads**: 8
- **Feed-Forward Network Hidden Size (\(d_{\text{ff}}\))**: 2048
- **Number of Layers (N)**: 6 for both encoder and decoder.

---

### **8. Advantages**
- **Parallelization**: The absence of recurrence allows for highly parallelizable operations.
- **Global Context**: Self-attention enables the model to learn dependencies across the entire sequence.
- **Scalability**: The architecture is well-suited for scaling to larger datasets and tasks.

---

The Transformer architecture is elegant in its simplicity and highly effective, forming the foundation of many state-of-the-art models in NLP and beyond. Let me know if you want further details on any component!