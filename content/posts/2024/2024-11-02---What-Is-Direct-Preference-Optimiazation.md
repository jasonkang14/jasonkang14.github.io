---
title: "What Is Direct Parameter Optimization(DPO)?"
date: "2024-11-02T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/what-is-direct-parameter-optimization"
category: "LLM"
tags:
  - "LLM"

description: "Investigating whether fine tuning can actually meet our needs"
---

# TL;DR
- Direct Preference Optimization (DPO) is a simplified method for aligning language models with human preferences by treating the model itself as an implicit reward model, avoiding the need for complex reinforcement learning. 
- Instead of a separate reward function, DPO uses a straightforward binary cross-entropy loss to directly optimize the model’s output towards preferred responses. This approach achieves performance on par with or better than traditional RLHF methods, like PPO, while being more computationally efficient and stable.


# Background 

I am working for a company where several of our affiliates operate under government oversight. This situation is somewhat unique to Korea, where certain major industrial facilities, such as refineries and power plants, are designated as "National Security Facilities." These facilities are required to comply with stringent security regulations enforced by the government.

One of the regulations enforced by the government mandates that data must remain within the company. This applies both literally and metaphorically. Physical documents cannot leave the company premises, and digital data must remain on internal servers. This presents a challenge when attempting to utilize Generative AI technologies, such as GPT or Claude, to enhance workplace efficiency, as invoking their APIs is considered equivalent to "data leaving the facility." Consequently, we are faced with the necessity to either develop our own Large Language Model or fine-tune an open-source model from HuggingFace to operate within our servers. Given the complexities and challenges associated with collecting and processing sufficient data to train a large language model from scratch, we have opted to fine-tune an existing model as a practical solution.

We became interested in [Direct Preference Optimization (DPO)](https://arxiv.org/abs/2305.18290) as we explored various methodologies for fine-tuning an existing large language model. This post seeks to analyze the paper and evaluate the advantages of employing DPO in the fine-tuning process.

# Addressing RLHF Limitations with Direct Preference Optimization

Direct Preference Optimization (DPO) was developed to address the limitations of Reinforcement Learning from Human Feedback (RLHF), a method employed by OpenAI to fine-tune their models for better adherence to human instructions. As the name implies, this approach requires human feedback on the outputs of a language model, with "rewards" assigned based on the model's performance. According to [a post from OpenAI in 2022](https://openai.com/index/instruction-following/), their labelers provided demonstrations of the desired model behavior on prompts submitted by customers to the OpenAI API. The labelers then ranked various outputs from the model, using this data to fine-tune GPT-3, thereby enhancing the model's ability to comprehend and follow human instructions.

This approach has evidently been successful, as demonstrated by ChatGPT's status as one of the most popular AI products globally. However, RLHF is inherently complex and can often result in instability. A fine-tuned model, which initially underwent unsupervised learning, must now optimize its performance based on a reward model that encapsulates human preferences. On the other hand, Direct Preference Optimization (DPO) offers a more stable and efficient alternative. By parameterizing the reward model to extract the optimal policy in closed form, DPO simplifies the fine-tuning process to a straightforward classification task. This eliminates the need for sampling from the language model during fine-tuning and reduces the complexity of hyperparameter tuning. As a result, DPO not only matches or exceeds the performance of traditional RLHF methods in aligning language models with human preferences but also simplifies implementation and training, making it a computationally lightweight and effective solution.




# What is Direct PReference Optimization(DPO)?

Instead of employing a separate reinforcement learning model as in RLHF, Direct Preference Optimization (DPO) leverages the language model itself as an implicit reward model. By utilizing binary cross-entropy, DPO transforms the task into a straightforward classification problem, thereby eliminating the multi-step processes inherent in RLHF. Given pairs of model-generated responses where one is preferred over the other, DPO directly optimizes the language model to increase the likelihood of preferred responses. 

Instead of calculating a reward explicitly, DPO leverages log-probability ratios from the model itself. For a given pair of responses (preferred and dispreferred), the model's log-probabilities are used to calculate an implicit reward. This is done by increasing the log probability of the preferred response relative to the dispreferred one, effectively treating the model’s outputs as a reward signal. This means that the model is trained to increase the likelihood of the preferred responses while decreasing the likelihood of the dispreferred responses.

![RLHF vs DPO](https://i.imgur.com/1nfpYSJ.png)

DPO leverages log-probability ratios from the model itself. For a given pair of responses (preferred and dispreferred), the model's log-probabilities are used to calculate an implicit reward. This is done by increasing the log probability of the preferred response relative to the dispreferred one, effectively treating the model’s outputs as a reward signal. DPO uses a closed-form solution to map these log-probabilities into a preference objective, making it possible to optimize the model parameters directly. This approach uses binary cross-entropy loss to maximize the probability of preferred responses and minimize that of dispreferred responses in a straightforward, supervised learning step. By doing this, DPO bypasses the multi-step RLHF process. The model does not explicitly decide how to update its parameters “on its own,” but it is updated based on a preference-guided loss function that effectively aligns its outputs with human preferences.

# DPO Experiment

## Experiment Overview
- **Tasks**: The experiments were conducted on three main tasks: sentiment-controlled text generation, summarization, and single-turn dialogue.
- **Models**: DPO was tested against several baseline methods, including supervised fine-tuning (SFT), Preferred Fine-Tuning (Preferred-FT), Unlikelihood Training, and RLHF with PPO.
- **Evaluation Metrics**: For some tasks, like sentiment, the ground truth was measurable, allowing a reward-KL trade-off analysis. For summarization and dialogue, win rates against baseline responses were computed, using GPT-4 as an evaluator to simulate human judgment.

## Task-Specific Experiments
### Controlled Sentiment Generation
- **Goal**: To control the sentiment of the language model’s responses, with a focus on positive sentiment in this case.
- **Setup**:
  - The IMDb dataset was used, with prompts being prefixes of movie reviews.
  - A sentiment classifier was employed as a reward function to distinguish preferred responses (positive sentiment) from dispreferred ones.
- **Baselines**:
  - The model trained with PPO-based RLHF was compared against DPO and other baselines, such as supervised fine-tuning and Preferred-FT.

- **Results**:
  - **Reward-KL Frontier**: DPO showed a more efficient reward-KL trade-off compared to PPO, achieving high sentiment alignment with lower KL divergence from the reference model.
  - **Performance**: DPO demonstrated a better balance between sentiment control and diversity of responses, outperforming PPO in terms of reward maximization with stability.

### Summarization
- **Goal**: To generate concise summaries of Reddit forum posts (using the TL;DR dataset).
- **Setup**:
  - A preference dataset from prior work was used, containing human preferences on summaries for specific posts.
- **Evaluation**:
  - Win rates were measured by comparing DPO-generated summaries to those from a reference model (usually SFT or Preferred-FT).
  - **Sampling Temperatures**: Different sampling temperatures were used to test the robustness of DPO. A higher temperature increases randomness in generation, potentially revealing more nuanced behavior across methods.
- **Results**:
  - DPO matched or outperformed PPO’s summarization quality in terms of win rate against human-preferred summaries, even when using GPT-4 to evaluate.
  - DPO maintained a high win rate across sampling temperatures, showing robustness that PPO lacked at higher temperatures.

![DPO works well with semtiment generation and summarization](https://i.imgur.com/aIoLzA5.png)

### Single-Turn Dialogue
- **Goal**: To produce helpful and engaging responses in single-turn dialogues, using the Anthropic “Helpful and Harmless” dataset.
- **Setup**:
  - The model aimed to respond to diverse questions in a way that maximized helpfulness and alignment with human preferences.
  - A pre-trained model (Pythia-2.8B) was fine-tuned with both DPO and PPO, and Preferred-FT served as a baseline for comparisons.
- **Evaluation**:
  - GPT-4 was used to evaluate win rates for DPO and baseline responses to test queries.
- **Results**:
  - DPO outperformed Preferred-FT and achieved a win rate similar to or better than the computationally intensive Best of N baseline (sampling multiple completions and choosing the best), but with less complexity and computational demand.
  - DPO consistently outperformed other methods in producing engaging, human-preferred responses.


# Conclusion 
- **Stability and Efficiency**: DPO’s single-step optimization was not only more stable than PPO but also simpler and more computationally efficient.
- **Alignment Performance**: DPO aligned with human preferences as well as, or better than, PPO in tasks involving sentiment, summarization, and dialogue.
Robustness: DPO maintained good performance across varying sampling temperatures, making it robust in different generation settings.

We are currently in the process of collecting data in order to fine-tune our model. If and only if it turns out to be successful, I will share the experience.