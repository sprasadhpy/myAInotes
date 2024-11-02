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

### Calculate **(U Σ') × Vᵀ**

**Adapted Matrix W':**

| 4.53 | 3.82 | 3.24 |
|------|------|------|
| 4.21 | 4.65 | 4.12 |
| 5.27 | 5.22 | 5.46 |


The finally adapted weight matrix **W'** reflects the injected preference vector. This modulates the model’s behavior to align with the user's preferences. By embedding the preference vector **λ = [0.8, 0.2]** into the singular values and also the model is now more aligned with the user’s preference for helpfulness (0.8) over conciseness (0.2).


######  for the beginners
Visualizing SVD:
Original Matrix: Think of a matrix as a transformation that stretches and rotates vectors.
Decomposition:
U: Defines the new set of orthogonal axes (left singular vectors).
Σ: Scales (stretches or shrinks) along each axis.
V^T: Defines how to rotate the data back to its original coordinate system.

A matrix is considered **low-rank** when its rank is smaller than the matrix’s total number of rows or columns. This indicates redundancy or dependency among the rows or columns.

If a matrix has rank **r** and dimensions **m × n** (where **m** represents the rows and **n** the columns):

- If **r < min(m, n)**, the matrix is a **low-rank matrix**.
- If **r = min(m, n)**, the matrix is a **full-rank matrix**.

Understanding the rank helps in analyzing the dimensional limitations and the dependency structure within the matrix.

****************************************************************************


Traditional AI has limitations -  which treats alignment as a single-objective optimization task focusing on one scalar goal (e.g., helpfulness or safety) with simple labels indicating "better" or "worse" outcomes. This approach is critiqued for oversimplifying human preferences and  which are often multi-dimensional and conflicting (e.g., helpfulness vs. conciseness). Panacea’s Multi-Dimensional Preference Optimization (MDPO) is introduced to tackle this issue by treating alignment as a multi-dimensional problem. Unlike single-objective methods MDPO optimizes multiple human preferences simultaneously, such as safety, humor, and formality and  finding Pareto-optimal solutions where no preference dimension can be improved without compromising another. This enables Panacea to recover the full Pareto front  of optimal solutions  offering a comprehensive set of trade-offs that better align model responses to the diverse preferences of human users. To be noted that  Panacea outperforms other approaches like AlignDiff and Rewarded Soups by achieving a more nuanced and customized alignment for complex human needs.



###### Comparison of AlignDiff, Rewarded Soups (RS), and Panacea

## (a) AlignDiff

-  AlignDiff operates in reinforcement learning (RL) environments and uses an attribute-conditioned diffusion model to align preferences within a multi-dimensional space. This model plans for optimal actions based on user preferences in dynamic RL settings.
- AlignDiff aims to address the challenge of aligning models with complex human preferences in dynamic and adaptable RL context  representing a recent step toward multi-dimensional alignment.

## (b) Rewarded Soups (RS)

- RS adopts a multi-policy strategy by training separate models for each preference dimension. For example, one model prioritizes helpfulness, while another focuses on conciseness. After training, RS linearly interpolates (combines) the parameters of these models to generate a customized model based on specific user preferences.
- RS does not encounter intermediate preference vectors during training, so it does not explicitly learn to handle nuanced or balanced preferences (e.g., equally prioritize both helpfulness and conciseness).
  - The interpolated model may not provide the best possible alignment due to limited exposure to combinations of preferences making it challenging to guarantee optimal solutions


How Panacea addresses the limitations of both AlignDiff and Rewarded Soups:

1. Panacea explicitly traverses the *preference simplex*, the multi-dimensional space that represents all possible trade-offs between preferences. This approach exposes Panacea to a variety of preference combinations during training, enabling it to learn how to handle not only extreme cases but also balanced preferences. The preference simplex is a geometric shape (e.g., a triangle in 2D, a tetrahedron in 3D) that contains all possible combinations of preferences. Panacea learns to navigate this shape and adapt to different trade-offs between preferences.

2: Panacea recovers the entire Pareto front the set of all possible Pareto-optimal solutions across varying preference combinations. This allows the model to generate responses that are more precisely aligned with individual user preferences.model has  the flexibility to cater to diverse user needs in a balanced way.



# Multi-Dimensional Preference Optimization (MDPO) for Aligning LLMs

MDPO is a method designed to align large language models (LLMs) to complex human preferences across multiple dimensions. Below are key concepts and equations explained in simplified terms.

Human preferences in interacting with AI systems are multi-dimensional, covering aspects like helpfulness, harmlessness, and humor. MDPO optimizes these preferences simultaneously, balancing potential conflicts (e.g., a more helpful response might be less concise).

The MDPO problem is to maximize performance across all preference dimensions. Mathematically:

`max J(π_θ) = (J₁(π_θ), J₂(π_θ), ..., Jₘ(π_θ))`

Where:
- `Jᵢ(π_θ)`: Performance measure for dimension `i` (e.g., helpfulness or harmlessness).
- `π_θ`: The policy, representing the LLM being trained, with parameters `θ`.
- `θ ∈ Θ`: The set of trainable parameters.
- `Π`: The policy space (all possible models).

Each preference dimension has a distinct objective function:

- **(a) SFT Objective `J_SFT,i(π_θ)`**:
   - Learns from labeled data `(x, y)`, maximizing the likelihood of generating correct output `y` given input `x`:
   
   `J_SFT,i(π_θ) = E_{(x,y) ∼ Dᵢ} [log π_θ(y|x)]`

   Here, `Dᵢ` is the dataset for dimension `i`, and `π_θ(y|x)` is the probability of generating `y` given `x`.

- **(b) RLHF Objective `J_RLHF,i(π_θ)`**:
   - Learns from rewards `rᵢ(x, y)`, with a KL-divergence term to keep the model close to a reference model `π_ref`:
   
   `J_RLHF,i(π_θ) = E_{x ∼ D} E_{y ∼ π_θ(⋅ | x)} [rᵢ(x, y)] - β D_KL[π_θ(⋅ | x) || π_ref(⋅ | x)]`

   Here, `rᵢ(x, y)` represents the reward for the response, and `β` is a scaling factor controlling deviation from the reference model.

- **(c) DPO Objective `J_DPO,i(π_θ)`**:
   - Compares two responses for the same input, aiming to prefer the "better" response while staying close to the reference model:
   
   `J_DPO,i(π_θ) = E_{(x, y_w, y_l) ∼ Dᵢ} [log σ(β (log π_ref(y_w | x) - log π_ref(y_l | x)))]`

   Here, `y_w` and `y_l` are the "better" and "worse" responses, respectively, and `σ` is the sigmoid function.

Because optimizing all dimensions perfectly is impossible (improving one might worsen another), MDPO seeks Pareto-optimal solutions.

A solution is Pareto-optimal if no other solution can improve one preference dimension without worsening another. Formally, for two solutions `θ_a` and `θ_b`:

`J(π_θ_a) ≻ J(π_θ_b)`

This means `θ_a` dominates `θ_b` if:
- `Jᵢ(π_θ_a) ≥ Jᵢ(π_θ_b)` for all dimensions `i`,
- and there exists at least one dimension `j` where `Jⱼ(π_θ_a) > Jⱼ(π_θ_b)`.

The Pareto Set (PS) is the set of all Pareto-optimal solutions, representing optimal trade-offs between preferences. The Pareto Front (PF) is the image of the Pareto set in objective space, showing trade-offs between performance measures.

Human preferences are represented by a preference vector `λ = (λ₁, ..., λₘ)`, where:
- `λᵢ ≥ 0`: Weight for dimension `i`.
- `Σ λᵢ = 1`: The total weight is normalized.

The preference simplex `Δₘ` is the space of all possible preference vectors, representing different trade-offs among preferences. MDPO seeks Pareto-optimal solutions for every possible preference vector.

For each training batch, Panacea samples a preference vector from the simplex and optimizes the model based on that vector. During inference, the model adapts to the user’s specified preference vector, ensuring Pareto-aligned behavior.

Panacea uses singular value decomposition (SVD) combined with low-rank adaptation (LoRA). The preference vector is embedded into the singular values of the SVD-decomposed weight matrices, scaled with learnable factors to adjust model behavior dynamically.


![Panacea Alignment Diagram](https://github.com/sprasadhpy/myAInotes/blob/shyaam_papers/Panacea1.png?raw=true)


This diagram compares single-objective alignment (left) and multi-dimensional alignment (right) for aligning AI model responses with human preferences across two dimensions (labeled here as **A** (e.g., helpfulness) and **B** (e.g., harmlessness)).

In single-objective alignment -  three different users rate two responses to a prompt, each with distinct preference weights. For eg, one rater might prioritize **A** (helpfulness) more (e.g., 0.7), while another prioritizes **B** (harmlessness) more (e.g., 0.6). The single-objective approach focuses on selecting a single preferred response based on either **A** or **B** alone. This creates misalignment with the diverse /&  multi-dimensional preferences of users, as it does not accommodate combined preferences. This method results in a "misaligned, conflicting, and singular" solution, lacking trade-offs between dimensions. Consequently, the solutions (represented by red crosses in the reward plot) fail to reach the Pareto front making them dominated solutions that do not capture optimal trade-offs between preferences **A** and **B**.

In multi-dimensional alignment - the model considers preference weights for both **A** and **B** simultaneously balancing each user’s preferences based on specific weightings to yield a coordinated response that respects both dimensions. This approach enables the selection of responses that balance both preferences. For eg, when a user prioritizes **A** (0.7) but also values **B** (0.3) and  the model can provide a response that respects this balance. Described as "aligned, coordinated, and diverse," this method employs Pareto optimality to ensure no preference can be improved without compromising another. Solutions (shown on the Pareto front in red) represent an optimal set where each point balances preferences **A** and **B** according to user-specific weights, covering the full spectrum of trade-offs.



### Theorem 4.1

Panacea recovers the entire Pareto front for both the Linear Scalarization (LS) and Tchebycheff (Tche) aggregation functions  under the following assumptions:

1.  Panacea with SVD-LoRA has sufficient flexibility to represent all preference vectors `λ ∈ Δₘ`. Specifically, for any preference vector `λ`, the policy `π_θ,λ` can optimize the corresponding aggregation functions (Equations (6) and (7)) to their maximum values.

2.  For a specific preference vector `λ`, the LLM policy space formed by all `π_θ,λ` can represent all possible categorical output distributions for responses.

By optimizing the Panacea objective function `E_{λ ∈ Δₘ} [g_agg(θ)]`, where `g_agg` can be either `g_LS` or `g_Tche`, the optimal policy found by Panacea can recover the entire Pareto front for almost every preference vector.

To put it simply this theorem states that Panacea is capable of adapting to a wide range of user preferences and can find the best possible trade-offs across different preference dimensions by effectively covering the entire set of optimal responses for any combination of user preferences.









