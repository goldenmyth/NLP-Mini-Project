# NLP Mini-Project (Module 3)
# Positional Fragility: Assessing Positional Sensitivity in Vector-Based Classifiers

This study investigates how positional information affects the accuracy and stability of text classification models under spatial shifts.
### **[Read the Full Report](/Report.md)**

---
## Project Idea & Assignment Tracks

This project combines Track 2 (Embedding-Based Classifiers) and Track 4 (Quantitative Studies). The primary focus is on analyzing how positional shifts influence model accuracy and representation stability.

The core idea:
While position is vital for text understanding, "naive" methods like Flattening create a rigid dependency on absolute coordinates. We compare standard semantic embeddings (Mean Pooling) with position-aware models (Standard Flatten and Explicit Pos-Aware) to quantify how much accuracy we lose when text shifts.

---
## Key Findings

Mean Pooling is the most stable: Keyword classification overrides word order in simple architectures (Zero degradation under shift).

Position causes Fragility: Explicit position models degrade twice as fast as non-positional ones under a 10-token shift.

Statistical Significance: McNemar's test (p<0.05) proves that positional displacement causes systematic structural errors, not random noise.

---
## Repository Structure

`Report.md` — Complete report with statistical test interpretations.

`solution/mini_project.ipynb` — The main Jupyter Notebook with data loading, training, and evaluations.

`solution/images/` - Visual assets.

`requirements.txt` — A list of Python dependencies.

---
## How to Run
    
### Option 1: Google Colab (Recommended)

The easiest way to run:
[Google Colab NLP Mini project](https://colab.research.google.com/drive/13F20cX8lhGzRwzb4h8-fSZfzqNZLoI6e?usp=sharing)

### Option 2: Local Installation

1. Clone the repository:
```
git clone https://github.com/goldenmyth/NLP-Mini-Project.git
cd NLP-Mini-Project
```

2. Install dependencies:

```
pip install -r requirements.txt
```

3. Open the notebook:
```
jupyter notebook solution/mini_project.ipynb
```
