# Positional Fragility: Quantifying Representation Stability under Contextual Shifts

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

**Length:** Sequence lengths range from 14 to 146 tokens (median: 39). **MAX_LEN = 59** was established via the 95th percentile to optimize memory and input dimensionality while capturing 95% of the data.

---
## Methodology & Training

**Architectures:** To ensure a fair comparison, all models were unified with a single hidden layer (32 neurons) and ReLU activation. This controlled for depth, focusing the analysis on the input aggregation method.

- **Pooling:** 100-dim mean vector input.
- **Flatten:** 5,900-dim flattened vector.
- **Position-Aware:** 5,959-dim vector (embeddings + indices).  

**Regularization:** To mitigate the "parameter explosion" in Flatten models (~190k params vs 3.2k in Pooling), we applied **AdamW (Weight Decay: 0.05)**, **BatchNorm**, and **Dropout (0.3–0.4)**.  

**Strategy:** Models were trained using CrossEntropyLoss (batch size 64) with **Early Stopping** (patience: 3) to capture weights at the point of optimal generalization, preventing excessive positional overfitting.

---
## Experimental Results & Analysis

**Shift Dynamics:** We measured accuracy as text shifted from 0 to 20 tokens. **Mean Pooling** remained nearly absolute , proving that for topic classification, keyword presence is more critical than specific coordinates. In contrast, both Flatten models showed rapid degradation.

[Accuracy Drop Chart](/solution/images/relative_accur_drop.png)
*Figure 1: Accuracy drop across different models during positional shift.*

Table 1 summarizes the degradation trends. The Position-Aware model exhibited the steepest decline (Slope: -0.540), indicating that explicit positional signals may anchor representations too strictly to their original indices.

*Table 1. Comparative Analysis of Representation Stability*

| Architecture | Slope (Sensitivity) | Volatility (Std) | Max Drop (%) | Total Loss Area |
| ------------ | ------------------- | ---------------- | ------------ | --------------- |
| **MEAN**     | -0.013              | 0.095            | 0.13         | 2.47            |
| **STD**      | -0.409              | 2.552            | 8.20         | 65.57           |
| **POS**      | -0.540              | 3.385            | 11.75        | 89.89           |

**Statistical Significance:** McNemar’s test confirmed the "positional advantage erosion" (`p<0.005`). While the **POS** model is initially competitive, its specific errors at Shift 20 are double those of the **STD** model (484 vs 228). This indicates that explicit indices may tether the model to a specific spatial context, substantially reducing its ability to generalize under displacement.

**Qualitative Error Trends:** Analysis of the Difference Matrix (Figure 2) at Shift 10 reveals specific linguistic vulnerabilities in the POS model compared to the baseline.

[Difference matrix](/solution/images/diff_matrix.png)
*Figure 2: Difference matrix POS minus Mean Pooling (shift = 10).*

- **Class-Specific Degradation:** The most substantial accuracy loss occurred in the Business (-10%) and Sports (-6%) categories.

- **Error Redistribution:** Misclassified Business samples were primarily redistributed to the World (+5%) and Sci/Tech (+5%) classes. Similarly, Sports news were increasingly mistaken for World news (+4%).

- **Interpretation:** These results suggest that positional displacement causes the model to lose track of specific semantic markers often found at the beginning of headlines (e.g., team names or financial entities). When these **"anchors"** shift to unseen coordinates, the fragile POS model defaults to broader categories, whereas the Mean Pooling model remains unaffected by the displacement.

---
## Conclusion

**Key Findings:** The experiment confirmed the **"positional fragility"** hypothesis. The **Position-Aware** model proved most vulnerable  to shifts, while Mean Pooling achieved the highest overall accuracy (89%). These results suggest that in simple FFNN architectures, the high parameter cost of positional encoding often leads to overfitting rather than improved semantic understanding. 

**Future Work:** 
- Conducting similar experiments on tasks where word order is vital to meaning (e.g., Natural Language Inference or Machine Translation) to evaluate the "price of order" in more complex linguistic scenarios.
- These results provide a baseline for our Final Project, where we will apply these insights to **Transformers**. We aim to implement **Positional Encoding Alignment** to enable KV-cache reuse and long-context stability without sacrificing the representational power of word order.