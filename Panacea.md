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



