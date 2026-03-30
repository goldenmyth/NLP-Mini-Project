# Positional Fragility: Assessing Positional Sensitivity in Vector-Based Classifiers

### **[Back to README](/README.md)**

---
## Abstract

This study evaluates the positional robustness of three architectures on the **AG News** dataset: invariant **Mean Pooling**, **Standard Flatten** (STD) (implicit position), and **Position-Aware** (POS) (explicit indices). Experimental results suggest a "positional fragility" phenomenon, where models incorporating positional features degrade significantly under contextual shifts (`p<0.001`, McNemar’s test). Mean Pooling demonstrated superior stability and accuracy due to its architectural bias toward semantic consistency over rigid ordering.

---
## Introduction & Problem Statement

While word order is a fundamental component of natural language, mapping embeddings to fixed indices via Flatten layers may establish a rigid dependency on absolute coordinates. When input text undergoes a spatial shift (e.g., due to prepended tokens), such models often fail to recognize semantic features at previously "unseen" positions. We analyze Feedforward Neural Networks (FFNN) over GloVe embeddings to quantify this effect across three configurations:

- **Mean Pooling:** Order-invariant baseline; expected to be perfectly stable but potentially less accurate.
- **Standard Flatten:** Implicitly encodes position via layer geometry; expected to show moderate fragility.
- **Position-Aware:** Appends normalized indices (0..1) to embeddings; expected to be the most accurate on static data but highly fragile under shifts.

---
## Data Processing & Exploratory Analysis

**Dataset:** We used the balanced **AG News** corpus (120k train / 7.6k test) across four categories. A 70k/10k train/split was employed. Given the 1:1:1:1 class ratio, **Accuracy** was selected as the primary representative metric.

**Pipeline:** A custom regex tokenizer was implemented to preserve abbreviations and contractions. Analysis of the GloVe (100d) vocabulary revealed a low **OOV rate of 1.53%**, primarily consisting of technical artifacts and possessives, indicating high semantic coverage.

**Length:** Sequence lengths range from 14 to 146 tokens (median: 39). We established a MAX_LEN of 59 (95th percentile). To prevent truncation during shift experiments, we utilized a fixed TOTAL_WINDOW of 80 tokens, allowing for a maximum shift of 21 tokens without losing words.

---
## Methodology & Training

**Architectures:** To ensure a fair comparison, all models were unified with a single hidden layer (32 neurons) and ReLU activation. This controlled for depth, focusing the analysis on the input aggregation method.

- **Pooling:** 100-dim mean vector input.
- **Flatten:** 8,000-dim flattened vector.
- **Position-Aware:** 8,080-dim vector (embeddings + indices).  

**Masked Mean Pooling:** To prevent padding tokens from distorting the average, we implemented a masking mechanism that averages only non-zero word vectors.

**Positional Grid:** For the POS model, we applied a continuous positional grid (0..1) across all 80 slots, ensuring even padding tokens maintained coordinate data.

**Regularization:** To mitigate the "parameter explosion" in Flatten models (~190k params vs 3.2k in Pooling), we applied **AdamW (Weight Decay: 0.05)**, **BatchNorm**, and **Dropout (0.3–0.4)**.  

**Strategy:** Models were trained using CrossEntropyLoss (batch size 64) with **Early Stopping** (patience: 3) to capture weights at the point of optimal generalization, preventing excessive positional overfitting.

---
## Experimental Results & Analysis

**Shift Dynamics:** We measured accuracy as text shifted from 0 to 20 tokens. **Mean Pooling** remained mathematically identical across all shifts (0.00% drop), proving that for topic classification, keyword presence is more critical than specific coordinates. In contrast, both Flatten models showed rapid degradation.

![Accuracy Drop Chart](/solution/images/relative_accur_drop.png)
*Figure 1: Accuracy drop across different models during positional shift.*

Table 1 summarizes the degradation trends. The Position-Aware model exhibited the steepest decline (Slope: -0.540), indicating that explicit positional signals may anchor representations too strictly to their original indices.

*Table 1. Comparative Analysis of Representation Stability*

| Model | Slope (Sensitivity) | Volatility (Std) | Max Drop (%) | Total Loss Area |
| :--- | :---: | :---: | :---: | :---: |
| MEAN | -0.000 | 0.000 | 0.00% | 0.00 |
| STD | -0.465 | 2.863 | 9.89% | 82.24 |
| POS | -0.417 | 2.564 | 8.83% | 77.11 |

**Statistical Significance:** As the Accuracy of POS and STD models were relatively close, McNemar’s test was crucial for verification. At all tested shift sizes (5, 10, 20), the `p-value < 0.05`, confirming that the POS model's superior resilience is statistically significant and not due to random weight initialization.

**Qualitative Error Trends:** Analysis of the Difference Matrix (Figure 2) at Shift 10 reveals specific linguistic vulnerabilities in the POS model compared to the baseline.

![Difference matrix](/solution/images/diff_matrix.png)
*Figure 2: Difference matrix POS minus Mean Pooling (shift = 10).*

- **Class-Specific Degradation:** The most substantial accuracy loss occurred in the Business (-24%) and Sports (-14%) categories.

- **Error Redistribution:** Misclassified Business samples were primarily redistributed to the World (+10%) and Sci/Tech (+15%) classes. Similarly, Sports news were increasingly mistaken for World news (+8%) and Sci/Tech (+7%).

- **Interpretation:** These results suggest that positional displacement causes the model to lose track of specific semantic markers often found at the beginning of headlines (e.g., team names or financial entities). When these **"anchors"** shift to unseen coordinates, the fragile POS model defaults to broader categories, whereas the Mean Pooling model remains unaffected by the displacement.

---
## Conclusion

**Key Findings:** The experiment confirmed that while positional awareness is a source of fragility in simple FFNNs, explicit encoding (POS) is significantly more robust than implicit learning (STD). Unexpectedly, Mean Pooling proved to be the most effective architecture for news classification, combining 89% accuracy with perfect positional invariance.

**Future Work:** 
- Conducting similar experiments on tasks where word order is vital to meaning (e.g., Natural Language Inference or Machine Translation) to evaluate the "price of order" in more complex linguistic scenarios.
- These results provide a baseline for our Final Project, where we will apply these insights to **Transformers**. We aim to implement **Positional Encoding Alignment** to enable KV-cache reuse and long-context stability without sacrificing the representational power of word order.
