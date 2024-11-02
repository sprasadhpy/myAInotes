# Panacea: Pareto Alignment via Preference Adaptation for LLMs
         [Read the paper](https://arxiv.org/pdf/2402.02030)


Summary of the paper :

Panacea paper addresses the limitations of traditional LLM alignment  which typically involves simplifying human preferences into scalar labels for optimizing responses. This approach does not capture the complexity of multi-dimensional human preferences  leading to misalignment ( yet to be  quantified - less literature) and biases in real-world applications


What are the major contributions 

1) The paper proposes reframing alignment as a  Multi-Dimensional Preference Optimization (MDPO) problem. Actually, this approach explicitly curates data for each dimension (e.g., helpfulness, harmlessness, humor), improving consistency and allowing for optimization across a broad spectrum of human preferences.

2) This paper aims to find solutions to the instances  where no single preference can be improved without sacrificing others. Instead of learning one solution the method seeks to recover the entire set of Pareto-optimal solutions ( similar to [Efficient Frontier](https://www.investopedia.com/terms/e/efficientfrontier.asp) by allowing the injection of a low-dimensional preference vector that dynamically guides the model’s behavior.

3) Panacea leverages singular value decomposition (SVD) and LoRA (low-rank adaptation) to efficiently adapt the model for different preferences. The preference vector is embedded in the model’s singular values providing fine-grained control of behavior.
