# Panacea: Pareto Alignment via Preference Adaptation for LLMs
         [Read the paper](https://arxiv.org/pdf/2402.02030)



The *Panacea* paper addresses the limitations of traditional LLM alignment, which typically involves simplifying human preferences into scalar labels to optimize responses. This approach fails to capture the complexity of multi-dimensional human preferences, leading to misalignment (yet to be fully quantified in literature) and biases in real-world applications.

Key contribution : 

The paper proposes reframing alignment as a Multi-Dimensional Preference Optimization (MDPO) problem. This approach explicitly curates data for each dimension (e.g., helpfulness, harmlessness, humor), improving consistency and allowing for optimization across a broad spectrum of human preferences.

The paper also aims to find solutions for instances where no single preference can be improved without sacrificing others. Instead of learning one solution, the method seeks to recover the entire set of Pareto-optimal solutions (similar to the [Efficient Frontier](https://www.investopedia.com/terms/e/efficientfrontier.asp)) by injecting a low-dimensional preference vector that dynamically guides the model’s behavior.

Panacea leverages singular value decomposition (SVD) and LoRA (low-rank adaptation) to adapt the model efficiently for different preferences. The preference vector is embedded in the model’s singular values, providing fine-grained control of behavior.

Panacea is compatible with various optimization techniques such as supervised fine-tuning (SFT), reinforcement learning from human feedback (RLHF), and direct preference optimization (DPO), allowing for scalability and robust performance.

Panacea is trained end-to-end using loss aggregation methods like linear scalarization (LS) and Tchebycheff.

Panacea was tested on challenging preference alignment problems with up to 10 dimensions, showcasing its ability to handle exponential growth in the Pareto set.

A major question raised by readers is how Panacea adapts a large language model’s behavior using SVD and LoRA, with a preference vector injected to control model behavior in real-time. Let’s walk through both a use case and a numerical example to clarify this process.


### Use Case  :  Real-Time User Preference Adaptation in Response Generation

What the task planned to do : 

To generate responses that adapt to specific user preferences (e.g., helpfulness, conciseness, harmlessness) in real-time (inference phase) by modifying the language model's behavior through user-defined preference vectors.

#### Key Process steps : 

1) Each user is assigned a unique **preference vector** that represents the importance of different response qualities.
   - **Example**:
     - **User A**: Prioritizes helpfulness (0.8) and harmlessness (0.2).
     - **User B**: Prioritizes conciseness (0.7) and harmlessness (0.3).

2)  **Singular Value Decomposition (SVD)** is applied to the model’s weight matrices divide them  into three components:
     - **U** (left singular matrix)
     - **Σ** (diagonal matrix with singular values) -  in this matrix  the user preference vector is injected. 
     - **V** (right singular matrix)
     - 
   - The **user's preference vector** is injected into the singular values (Σ), adjusting how the model prioritizes qualities in its responses.
   - **Learnable scaling factors** fine-tune the influence of the preference vector to achieve the desired response characteristics.

3) Based on the preference vector- 
     - **User A (Helpfulness-focused)**: Receives a detailed and elaborate response, providing actionable steps and comprehensive information.
     - **User B (Conciseness-focused)**: Receives a short, to-the-point response with only essential details.

The system dynamically adjusts the singular values according to the preference vector during inference, allowing the model to switch between different response styles **without retraining**.


### Numerical Example :Injection Process of Panacea using SVD and LoRA

Suppose we have a weight matrix **W** from one layer of the model. Let’s assume it’s a simple \( 3 \times 3 \) matrix:

**Matrix W:**

| 4 | 1 | 3 |
|---|---|---|
| 2 | 5 | 6 |
| 7 | 8 | 9 |

### Singular Value Decomposition (SVD)

We apply Singular Value Decomposition (SVD) to decompose this matrix into three matrices: **U**, **Σ**, and **Vᵀ**:

\[
W = U \, Σ \, Vᵀ
\]

- **U**: orthogonal matrix (captures the left singular vectors),
- **Σ**: diagonal matrix (captures the singular values),
- **Vᵀ**: orthogonal matrix (captures the right singular vectors).

Let’s assume that after applying SVD, we obtain:

**Matrix U:**

| 0.58 | -0.58 | 0.58 |
|------|-------|------|
| 0.43 | 0.71  | 0.57 |
| 0.69 | 0.0   | -0.69|

**Matrix Σ:**

| 12  | 0   | 0   |
|-----|-----|-----|
| 0   | 4   | 0   |
| 0   | 0   | 2   |

**Matrix Vᵀ:**

| 0.58 | 0.58 | 0.58 |
|------|------|------|
| -0.58| 0.71 | 0.57 |
| 0.58 | -0.0 | -0.69|


Suppose we have a preference vector **λ** representing user preferences. For instance, let’s assume **λ** has two dimensions for "helpfulness" and "conciseness" with values:

\[
λ = [0.8, 0.2]
\]

Panacea injects this preference vector into the singular values matrix **Σ** using a scaling factor **s** to control the influence of the preference vector. Assume **s = 0.5**.

To modify **Σ**, we inject **λ** into the second and third positions:

**Modified Σ':**

| 12  | 0   | 0   |
|-----|-----|-----|
| 0   | 0.4 | 0   |
| 0   | 0   | 0.1 |


With the modified **Σ'** Panacea reconstruct the adapted weight matrix **W'** by multiplying **U**, **Σ'**, and **Vᵀ**:

\[
W' = U \, Σ' \, Vᵀ
\]

### Calculate **U × Σ'**

**Result of U Σ':**

| 6.96  | -0.23 | 0.058  |
|-------|-------|--------|
| 5.16  | 0.28  | 0.057  |
| 8.28  | 0.0   | -0.069 |

###Calculate **(U Σ') × Vᵀ**

**Adapted Matrix W':**

| 4.53 | 3.82 | 3.24 |
|------|------|------|
| 4.21 | 4.65 | 4.12 |
| 5.27 | 5.22 | 5.46 |


The finally adapted weight matrix **W'** reflects the injected preference vector. This modulates the model’s behavior to align with the user's preferences. By embedding the preference vector **λ = [0.8, 0.2]** into the singular values and also the model is now more aligned with the user’s preference for helpfulness (0.8) over conciseness (0.2).













