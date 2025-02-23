---
title: "Understanding Temperature and Top-p Sampling in Large Language Models" 
date: "2025-02-23T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/understanding-temperature-and-top-p-sampling-in-large-language-models"
category: "LLM"
tags:
  - "LLM"

description: "Explore the intricacies of temperature and top-p sampling in Large Language Models, understanding their impact on text generation and practical applications."
---

# Understanding Temperature and Top-p Sampling in Large Language Models: A Deep Dive

When working with Large Language Models (LLMs), two parameters often confound even experienced practitioners: temperature and top-p sampling. These parameters fundamentally control how LLMs generate text, yet their implications aren't always intuitive. In this post, we'll demystify these concepts and explore how they shape LLM outputs in practice.

## The Foundation: Probability Distributions in LLMs

Before diving into sampling methods, let's understand what we're sampling from. When an LLM processes a prompt, it generates a probability distribution over its entire vocabulary for the next token. For example, given the prompt "The capital of France is", the model might assign:

- "Paris" → 0.85 probability
- "Lyon" → 0.05 probability
- "London" → 0.02 probability
- Other tokens → remaining probability

Without any sampling parameters, the model would simply choose the token with the highest probability (greedy decoding). However, this often leads to repetitive and deterministic outputs. This is where temperature and top-p sampling come in.

## Temperature: Controlling Randomness

Temperature is perhaps the most widely known sampling parameter, typically ranging from 0.0 to 2.0. It works by modifying the probability distribution before sampling occurs. The mathematical transformation is:

$$ 
\text{new\_probabilities} = \exp\left(\frac{\log(\text{original\_probabilities})}{\text{temperature}}\right)
$$  


Let's break down what different temperature values do:

### Understanding Temperature Through Examples

Let's explore how different temperature values transform probability distributions through concrete examples. Consider an LLM generating the next word after "The weather today is":

Initial probability distribution:
- "sunny" → 0.50
- "cloudy" → 0.30
- "rainy" → 0.15
- "stormy" → 0.05

Let's see how different temperatures affect these probabilities:

### Temperature = 0 (The Ice Age)
At temperature 0, we get a deterministic output:
```python
# T = 0 effectively turns probabilities into:
"sunny" → 1.0 (100%)
"cloudy" → 0.0 (0%)
"rainy" → 0.0 (0%)
"stormy" → 0.0 (0%)
```

The model will always choose "sunny" as it has the highest probability. This makes outputs completely deterministic and is ideal for:
- Solving mathematical problems
- Writing code
- Generating structured data

### Temperature = 0.5 (Cool and Controlled)
Let's calculate the new probabilities using the formula: p_new = exp(log(p)/0.5)
```python
"sunny" → exp(log(0.50)/0.5) ≈ 0.70 (70%)
"cloudy" → exp(log(0.30)/0.5) ≈ 0.22 (22%)
"rainy" → exp(log(0.15)/0.5) ≈ 0.07 (7%)
"stormy" → exp(log(0.05)/0.5) ≈ 0.01 (1%)
```

Notice how this temperature sharpens the distribution, making high-probability tokens even more likely while still maintaining some randomness. This works well for:
- Technical writing
- Structured creative tasks
- Semi-formal conversation

### Temperature = 1.0 (Room Temperature)
At temperature 1.0, we maintain the original probabilities:
```python
"sunny" → 0.50 (50%)
"cloudy" → 0.30 (30%)
"rainy" → 0.15 (15%)
"stormy" → 0.05 (5%)
```

This preserves the model's original probability assessments, providing a balanced mix of creativity and coherence. Ideal for:
- General conversation
- Creative writing with constraints
- Summarization tasks

### Temperature = 2.0 (The Heat Is On)
Let's calculate: p_new = exp(log(p)/2.0)
```python
"sunny" → exp(log(0.50)/2.0) ≈ 0.35 (35%)
"cloudy" → exp(log(0.30)/2.0) ≈ 0.27 (27%)
"rainy" → exp(log(0.15)/2.0) ≈ 0.19 (19%)
"stormy" → exp(log(0.05)/2.0) ≈ 0.11 (11%)
```

Notice how the probabilities are much closer together now. The model is more likely to choose lower-probability tokens, leading to more diverse and potentially surprising outputs. This setting works for:
- Brainstorming sessions
- Poetry generation
- Exploring alternative perspectives

### Visualizing the Impact

To understand the mathematical transformation more deeply, let's look at what happens to probability ratios. Consider the ratio between "sunny" and "stormy" at different temperatures:

Original ratio: 0.50/0.05 = 10:1

- At T=0.5: 0.70/0.01 ≈ 70:1 (more extreme)
- At T=1.0: 0.50/0.05 = 10:1 (unchanged)
- At T=2.0: 0.35/0.11 ≈ 3.2:1 (more balanced)

This demonstrates how temperature controls the "sharpness" of the probability distribution. Lower temperatures amplify differences between probabilities, while higher temperatures smooth them out.

## Top-p Sampling: A More Nuanced Approach

While temperature modifies the entire probability distribution, top-p sampling (also known as nucleus sampling) takes a different approach. Let's explore this through detailed examples to understand how it works in practice.

### Understanding Top-p Through Examples

Let's consider an LLM generating the next word for the prompt "The capital of France is". Here's our initial probability distribution:

```python
Initial probabilities:
"Paris" → 0.60
"Lyon" → 0.15
"Marseille" → 0.10
"London" → 0.05
"Berlin" → 0.04
"Madrid" → 0.03
"Rome" → 0.02
"Other tokens" → 0.01
```

Let's see how different top-p values affect token selection:

### Top-p = 0.9 (Standard Setting)
Let's walk through the calculation:

1. Sort by probability (already done above)
2. Calculate cumulative probabilities:
```python
"Paris" → 0.60 (cumulative: 0.60)
"Lyon" → 0.15 (cumulative: 0.75)
"Marseille" → 0.10 (cumulative: 0.85)
"London" → 0.05 (cumulative: 0.90) ← Cutoff point
"Berlin" → 0.04 (excluded)
"Madrid" → 0.03 (excluded)
"Rome" → 0.02 (excluded)
"Other tokens" → 0.01 (excluded)
```

The model will only sample from the first four tokens, maintaining their relative probabilities. To get the final sampling probabilities, we renormalize within our selected tokens:

```python
Final sampling probabilities:
"Paris" → 0.60/0.90 ≈ 0.667 (66.7%)
"Lyon" → 0.15/0.90 ≈ 0.167 (16.7%)
"Marseille" → 0.10/0.90 ≈ 0.111 (11.1%)
"London" → 0.05/0.90 ≈ 0.056 (5.5%)
```

### Top-p = 0.75 (More Conservative)
Let's see a more restrictive setting:

```python
"Paris" → 0.60 (cumulative: 0.60)
"Lyon" → 0.15 (cumulative: 0.75) ← Cutoff point
"Marseille" → 0.10 (excluded)
[remaining tokens excluded]

Final sampling probabilities:
"Paris" → 0.60/0.75 = 0.80 (80%)
"Lyon" → 0.15/0.75 = 0.20 (20%)
```

Notice how this makes the model more likely to choose "Paris" compared to the 0.9 setting.

### Top-p = 0.50 (Highly Conservative)
In this case:

```python
"Paris" → 0.60 (cumulative: 0.60) ← Cutoff point exceeded
[all other tokens excluded]

Final sampling probabilities:
"Paris" → 1.0 (100%)
```

This becomes equivalent to greedy sampling since only the highest-probability token makes it into our sampling pool.

### Combining Temperature and Top-p

When using both parameters, the order of operations matters:

1. First, apply temperature to modify the initial distribution
2. Then, apply top-p sampling to the modified distribution

Let's see an example with temperature = 2.0 and top-p = 0.9:

```python
Original → After Temperature → After Top-p (cumulative)
"Paris": 0.60 → 0.35 → 0.35 (0.35)
"Lyon": 0.15 → 0.19 → 0.19 (0.54)
"Marseille": 0.10 → 0.15 → 0.15 (0.69)
"London": 0.05 → 0.11 → 0.11 (0.80)
"Berlin": 0.04 → 0.09 → 0.09 (0.89)
"Madrid": 0.03 → 0.07 → 0.01 (0.90)
[remaining tokens excluded]
```

This demonstrates how temperature can first flatten the distribution, allowing more tokens to be included in the top-p sampling pool, leading to more diverse outputs while still maintaining coherence.

### Key Advantages of Top-p Sampling

This approach offers several benefits:
- Dynamically adapts to the model's confidence (uses fewer tokens when the model is very confident)
- Prevents sampling from the long tail of very unlikely tokens
- Maintains coherence while allowing appropriate levels of creativity
- Computationally efficient since we only need to consider a subset of tokens

## Practical Implications: When to Use What?

The choice between temperature and top-p sampling (or using both) depends on your specific use case:

### High-Precision Tasks
For tasks requiring accuracy and consistency:
- Temperature: 0.0-0.3
- Top-p: 0.1-0.3
Example use cases: Code generation, factual Q&A, structured data generation

### Balanced Generation
For general-purpose text generation:
- Temperature: 0.7-0.9
- Top-p: 0.9
Example use cases: Chat responses, content generation, summarization

### Creative Tasks
For maximizing creativity:
- Temperature: 1.0-1.5
- Top-p: 0.95-1.0
Example use cases: Storytelling, poetry, ideation

## Understanding Top-k Sampling

While temperature modifies probabilities and top-p works with cumulative probability mass, top-k sampling takes perhaps the most straightforward approach: it simply keeps a fixed number (k) of the most likely tokens and discards the rest.

### How Top-k Works

Let's use our previous example of completing "The capital of France is":

```python
Initial probabilities:
"Paris" → 0.60
"Lyon" → 0.15
"Marseille" → 0.10
"London" → 0.05
"Berlin" → 0.04
"Madrid" → 0.03
"Rome" → 0.02
"Other tokens" → 0.01
```

With top-k = 3, we would:
1. Keep only the 3 most likely tokens:
```python
"Paris" → 0.60
"Lyon" → 0.15
"Marseille" → 0.10
```

2. Renormalize the probabilities to sum to 1:
```python
"Paris" → 0.60 / 0.85 ≈ 0.706 (70.6%)
"Lyon" → 0.15 / 0.85 ≈ 0.176 (17.6%)
"Marseille" → 0.10 / 0.85 ≈ 0.118 (11.8%)
```

### Implementation Example

Here's how we can implement top-k sampling:

```python
def apply_top_k(
    probs: Dict[str, float], 
    k: int
) -> Dict[str, float]:
    """
    Apply top-k sampling to a probability distribution.
    
    Args:
        probs (Dict[str, float]): Original token probabilities
        k (int): Number of top tokens to keep
        
    Returns:
        Dict[str, float]: Modified probability distribution
    """
    # Sort tokens by probability
    sorted_probs = sorted(
        probs.items(), 
        key=lambda x: x[1], 
        reverse=True
    )
    
    # Keep only top k tokens
    top_k_tokens = sorted_probs[:k]
    
    # Create new distribution with only selected tokens
    new_probs = dict(top_k_tokens)
    
    # Renormalize probabilities
    normalization = sum(new_probs.values())
    return {
        token: prob / normalization 
        for token, prob in new_probs.items()
    }

# Example showing different k values
distribution = {
    "Paris": 0.60,
    "Lyon": 0.15,
    "Marseille": 0.10,
    "London": 0.05,
    "Berlin": 0.04,
    "Madrid": 0.03,
    "Rome": 0.02,
    "Other": 0.01
}

k_values = [2, 3, 4]
for k in k_values:
    modified = apply_top_k(distribution, k)
    print(f"\nTop-k {k}:")
    for token, prob in modified.items():
        print(f"{token}: {prob:.3f}")
```

### Comparing Sampling Methods

Let's examine how top-k compares to our other sampling methods:

1. **Top-k vs Temperature**
   - Temperature: Modifies the entire distribution, making it sharper or flatter
   - Top-k: Simply cuts off all but the k most likely tokens
   - Temperature can work with top-k, modifying probabilities before the cutoff

2. **Top-k vs Top-p**
   - Top-k: Uses a fixed number of tokens
   - Top-p: Adapts the number of tokens based on probability mass
   - Top-p is generally more flexible and context-aware

### Limitations of Top-k

Top-k sampling has some important limitations:

1. **Fixed Cutoff**: Using a fixed k regardless of the probability distribution can be problematic:
   ```python
   # Case 1: Sharp distribution
   "the" → 0.9
   "a" → 0.05
   "an" → 0.03
   "that" → 0.02
   
   # Case 2: Flat distribution
   "blue" → 0.3
   "red" → 0.28
   "green" → 0.27
   "yellow" → 0.15
   ```
   With k=2, we'd keep two tokens in both cases, even though this makes more sense in Case 1 than Case 2.

2. **Arbitrary Cutoffs**: Similar tokens might be treated very differently if they fall just above or below k:
   ```python
   k = 3:
   "dog" → 0.4 (kept)
   "cat" → 0.3 (kept)
   "puppy" → 0.2 (kept)
   "kitten" → 0.1 (discarded despite similarity to kept tokens)
   ```

### When to Use Top-k

Despite its limitations, top-k can be useful in specific scenarios:

1. When you want precise control over the maximum number of tokens to consider
2. In systems with computational constraints where evaluating more tokens is expensive
3. Combined with temperature to prevent sampling from the long tail while still allowing probability modification

### Best Practices

1. **Choosing k**: Start with k values between 20-100 for general text generation
2. **Combining Methods**: Consider using top-k with temperature or combining all three methods
3. **Testing**: Always test different k values with your specific use case and model

The code implementation lets you experiment with different k values and see their effects on the probability distribution. Try running it with different initial distributions and k values to build intuition about how top-k sampling behaves.

## Implementation Examples

Let's implement both temperature and top-p sampling in Python, with detailed examples showing how they transform probability distributions. We'll start with some utility functions and then implement each sampling method.

### Utility Functions

```python
import numpy as np
from typing import List, Dict, Tuple

def softmax(logits: np.ndarray) -> np.ndarray:
    """
    Convert logits to probabilities using softmax function.
    
    Args:
        logits (np.ndarray): Raw logit scores from the model
    
    Returns:
        np.ndarray: Probability distribution summing to 1
    """
    exp_logits = np.exp(logits - np.max(logits))  # Subtract max for numerical stability
    return exp_logits / exp_logits.sum()

def create_vocab_distribution(vocab: List[str], probs: List[float]) -> Dict[str, float]:
    """
    Create a dictionary mapping vocabulary tokens to their probabilities.
    
    Args:
        vocab (List[str]): List of tokens
        probs (List[float]): Corresponding probabilities
        
    Returns:
        Dict[str, float]: Mapping of tokens to probabilities
    """
    return dict(zip(vocab, probs))
```

### Temperature Sampling Implementation

```python
def apply_temperature(
    probs: Dict[str, float], 
    temperature: float
) -> Dict[str, float]:
    """
    Apply temperature scaling to a probability distribution.
    
    Args:
        probs (Dict[str, float]): Original token probabilities
        temperature (float): Temperature parameter (> 0)
        
    Returns:
        Dict[str, float]: Modified probability distribution
    """
    if temperature == 0:
        # For temperature 0, return 1.0 for highest prob token, 0 for others
        max_token = max(probs.items(), key=lambda x: x[1])[0]
        return {token: float(token == max_token) for token in probs}
    
    # Convert probabilities to log space
    log_probs = {token: np.log(p) for token, p in probs.items()}
    
    # Apply temperature scaling
    scaled_log_probs = {token: lp / temperature for token, lp in log_probs.items()}
    
    # Convert back to probabilities
    max_log_p = max(scaled_log_probs.values())  # For numerical stability
    exp_probs = {
        token: np.exp(lp - max_log_p) 
        for token, lp in scaled_log_probs.items()
    }
    
    # Normalize to get final probabilities
    normalization = sum(exp_probs.values())
    return {token: p / normalization for token, p in exp_probs.items()}

# Example usage:
vocab = ["Paris", "Lyon", "Marseille", "London", "Berlin"]
probs = [0.6, 0.15, 0.1, 0.05, 0.1]
distribution = create_vocab_distribution(vocab, probs)

# Try different temperatures
temperatures = [0.5, 1.0, 2.0]
for temp in temperatures:
    modified = apply_temperature(distribution, temp)
    print(f"\nTemperature {temp}:")
    for token, prob in modified.items():
        print(f"{token}: {prob:.3f}")
```

### Top-p (Nucleus) Sampling Implementation

```python
def apply_top_p(
    probs: Dict[str, float], 
    p: float
) -> Dict[str, float]:
    """
    Apply top-p (nucleus) sampling to a probability distribution.
    
    Args:
        probs (Dict[str, float]): Original token probabilities
        p (float): Cumulative probability threshold (0 < p ≤ 1)
        
    Returns:
        Dict[str, float]: Modified probability distribution
    """
    # Sort tokens by probability
    sorted_probs = sorted(
        probs.items(), 
        key=lambda x: x[1], 
        reverse=True
    )
    
    # Calculate cumulative probabilities
    cumulative = 0
    keep_tokens = []
    for token, prob in sorted_probs:
        cumulative += prob
        keep_tokens.append(token)
        if cumulative >= p:
            break
    
    # Create new distribution with only selected tokens
    new_probs = {
        token: probs[token] for token in keep_tokens
    }
    
    # Renormalize probabilities
    normalization = sum(new_probs.values())
    return {
        token: prob / normalization 
        for token, prob in new_probs.items()
    }

# Example usage:
p_values = [0.9, 0.75, 0.5]
for p in p_values:
    modified = apply_top_p(distribution, p)
    print(f"\nTop-p {p}:")
    for token, prob in modified.items():
        print(f"{token}: {prob:.3f}")
```

### Combining Temperature and Top-p

```python
def sample_token(
    probs: Dict[str, float], 
    temperature: float = 1.0, 
    top_p: float = 1.0
) -> str:
    """
    Sample a token using both temperature and top-p sampling.
    
    Args:
        probs (Dict[str, float]): Original token probabilities
        temperature (float): Temperature parameter (> 0)
        top_p (float): Cumulative probability threshold (0 < p ≤ 1)
        
    Returns:
        str: Sampled token
    """
    # First apply temperature
    temp_probs = apply_temperature(probs, temperature)
    
    # Then apply top-p
    final_probs = apply_top_p(temp_probs, top_p)
    
    # Sample from the final distribution
    tokens = list(final_probs.keys())
    probabilities = list(final_probs.values())
    return np.random.choice(tokens, p=probabilities)

# Example usage showing multiple samples:
print("\nSampling with temperature=0.8 and top_p=0.9:")
for _ in range(5):
    token = sample_token(distribution, temperature=0.8, top_p=0.9)
    print(f"Sampled token: {token}")
```

### Putting It All Together: A Complete Example

```python
def analyze_sampling_parameters(
    vocab: List[str],
    probs: List[float],
    temperature: float,
    top_p: float,
    n_samples: int = 1000
) -> Dict[str, int]:
    """
    Analyze the effects of sampling parameters by generating multiple samples.
    
    Args:
        vocab (List[str]): List of tokens
        probs (List[float]): Original probabilities
        temperature (float): Temperature parameter
        top_p (float): Top-p threshold
        n_samples (int): Number of samples to generate
        
    Returns:
        Dict[str, int]: Count of how many times each token was sampled
    """
    distribution = create_vocab_distribution(vocab, probs)
    samples = [
        sample_token(distribution, temperature, top_p)
        for _ in range(n_samples)
    ]
    return dict(Counter(samples))

# Example usage:
vocab = ["Paris", "Lyon", "Marseille", "London", "Berlin"]
probs = [0.6, 0.15, 0.1, 0.05, 0.1]

# Try different parameter combinations
parameter_sets = [
    (1.0, 1.0),  # Baseline
    (0.5, 1.0),  # Lower temperature
    (1.0, 0.9),  # Top-p only
    (0.5, 0.9),  # Both
]

for temp, p in parameter_sets:
    counts = analyze_sampling_parameters(
        vocab, probs, temp, p, n_samples=1000
    )
    print(f"\nTemperature={temp}, Top-p={p}")
    for token, count in counts.items():
        print(f"{token}: {count/1000:.3f}")
```

This implementation provides a complete framework for experimenting with temperature and top-p sampling. The code is designed to be educational and includes detailed comments explaining each step. You can use it to:

1. Understand how each parameter transforms probabilities
2. Visualize the effects of different parameter combinations
3. Generate samples to see the practical impact
4. Analyze the distribution of sampled tokens

The example usage sections demonstrate how to use each function and help build intuition about how these parameters affect token selection in practice.

## Conclusion

Understanding temperature and top-p sampling is crucial for getting the most out of LLMs. These parameters offer fine-grained control over the balance between creativity and determinism in model outputs. By carefully tuning these parameters based on your specific use case, you can significantly improve the quality and appropriateness of generated content.

Remember that there's no one-size-fits-all setting – experimentation is key to finding the right balance for your specific application. Consider creating a systematic testing process to evaluate different parameter combinations for your particular use case.

## Additional Resources

For those interested in diving deeper into this topic, I recommend exploring:
- The original Nucleus Sampling paper by Holtzman et al.
- Practical implementations in popular LLM frameworks
- Real-world case studies of parameter tuning in production systems

What sampling strategies have you found most effective in your LLM applications? Share your experiences and insights in the comments below!