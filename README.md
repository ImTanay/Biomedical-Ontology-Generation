# Benchmarking Resource-Efficient LLMs for Research Topic Ontology Generation in the Biomedical Field

This repository offers datasets and scripts to fine-tune and evaluate LLMs for identifying semantic relationships between biomedical research topic pairs. It aims to advance the automated creation and evolution of research topic ontologies within the biomedical domain.

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

---

## 📜 Abstract
Knowledge Organization Systems like ontologies and taxonomies are fundamental for structuring scientific knowledge, as they facilitate the efficient classification, dissemination, and retrieval of information. However, the creation and maintenance of such ontologies are expensive and time-consuming tasks, usually requiring the coordinated effort of multiple domain experts. Consequently, the development of domain-specific ontologies remains predominantly a manual process. In this study, we investigate the capability of five small, open-source LLMs (up to 9 billion parameters) to identify semantic relationships among research topics within the biomedical domain. The models were evaluated under three distinct conditions: **standard prompting**, **chain-of-thought prompting**, and **fine-tuning**. To support this analysis, we introduce **MeSH-Rel-4K**, a dataset consisting of 4,000 relationships extracted from the Medical Subject Headings ([MeSH](https://www.nlm.nih.gov/mesh/meshhome.html)) vocabulary. Our experiments demonstrate that targeted fine-tuning increases the average F1-score by 34.1 percentage points, providing an accurate, automated methodology for the construction and evolution of multi-domain scientific ontologies.

---

## 📂 Repository Structure

### 📄 `datasets/`
This folder contains the **MeSH-Rel-4K** dataset used in this study. Access it [here](./datasets).

## 📊 Dataset Construction: MeSH-Rel-4K

**MeSH-Rel-4K** is a dataset of 4,000 semantic relationships derived from the January 2025 release of the Medical Subject Headings ([MeSH](https://www.nlm.nih.gov/mesh/meshhome.html)). 

The dataset was systematically constructed to ensure high-quality semantic relationships. Below is a detailed breakdown of the dataset construction process:

| **Domain** | **Dataset** | **broader** | **narrower** | **same-as** | **other** | **Total** |
|-------------------|------------------|-------------|--------------|-----------------|-----------|-----------|
| **Biomedical** | **MeSH-Rel-4K** | 1,000       | 1,000        | 1,000           | 1,000     | 4,000     |

### 🔍 Dataset Details
- **Source**: Medical Subject Headings ([MeSH](https://www.nlm.nih.gov/mesh/meshhome.html))
- **Composition**:
    - 1,000 relationships for `broader`, derived from `mesh:broaderDescriptor`.
    - 1,000 relationships for `narrower`, derived from `mesh:broaderDescriptor`.
    - 1,000 relationships for `same-as`, derived from `mesh:relatedConcept`.
    - 1,000 unrelated topic pairs to populate the `other` category.

### 📂 Dataset Splits
The **MeSH-Rel-4K** dataset was partitioned into training, validation, and test sets using a 7:1:2 ratio.

| **Dataset** | **Train** | **Validation** | **Test** | **Total** | **Percentage** |
|-------------------|-----------|----------------|----------|-----------|----------------|
| **MeSH-Rel-4K** | 2,800     | 400            | 800      | 4,000     | 100%           |

This structured dataset enables robust evaluation of LLMs for semantic relationship classification within the biomedical discipline.

---

### 🖥 `code/`
This folder contains the **scripts** for identifying semantic relationships between research topic pairs. The scripts implement various strategies, including **standard prompting (`STD_prompting`)**, **chain-of-thought prompting (`CoT-2-way_prompting`)**, and **fine-tuning (`fine_tuning`)**. Access them [here](./code)

#### Task Overview
The task addressed in this study involves identifying the semantic relationship that holds between a given pair of research topics, denoted as $t_A$ and $t_B$. This task is formalised as a single-label, multi-class classification problem, where each input pair $(t_A, t_B)$ is assigned to exactly one of the following mutually exclusive categories:

- **broader**: $t_A$ is a parent topic of $t_B$. For example, *plants* is broader than *agricultural crops*.
- **narrower**: $t_A$ is a child topic of $t_B$. For example, *pneumocystis infections* is contained within *fungal diseases*.
- **same-as**: $t_A$ and $t_B$ can be used interchangeably to represent the same research concept. For example, *taste dysfunction* and *taste disorders*.
- **other**: $t_A$ and $t_B$ are unrelated or when none of the above applies. It rather provides a mechanism for the classifier to label negative examples.

#### Experiment Strategies

The experimental strategies in this study are categorised into two prompting-based approaches and one fine-tuning-based approach:

1. **Prompting-based Strategies**:
    - **Standard Prompting**: Prompts are generated for each pair of research topics using a predefined template. 
    - **CoT, Two-way Strategy**: Employs a two-stage sequential prompting mechanism. The initial prompt instructs the model to define both topics, construct a sentence incorporating them, and reflect upon their potential semantic relationship. To ensure robustness, the two topics are then swapped, and the entire process is repeated.

      *Comprehensive details regarding the prompt templates are available at: https://github.com/ImTanay/LLM-Automatic-Ontology-Generation*

2. **Fine-tuning-based Strategies**:
    - **Domain-specific Fine-Tuning**: The five open-source LLMs were fine-tuned on the MeSH-Rel-4K dataset. We used Unsloth, which supports 4-bit quantised training via BitsAndBytes, substantially reducing memory consumption and permitting training on consumer-grade GPUs.

    #### Prompt Template
    A standardised base prompt template was employed during fine-tuning:

    ```Classify the relationship between '[TOPIC-A]' and '[TOPIC-B]'```

#### The table below provides an overview of the 5 LLMs used in our experiments
It includes the **Model** name, the alias adopted in this study, the number of trainable **Parameters**, the context **Window** size, and the rank and scaling factor of the low-rank adaptation matrices used in LoRA (**r** and **alpha**).

| **Model** | **Alias** | **Parameters** | **Window** | **r** | **alpha** | **Link** |
|-------------------------------|-------------------|----------------|------------|-------|-----------|---------------------------------------------------------------------------|
| mistral-7b-instruct-v0.3      | mistral-7b        | 7.25B          | 32K        | 16    | 16        | [mistral-7b-instruct-v0.3](https://huggingface.co/unsloth/mistral-7b-instruct-v0.3) |
| Llama-3.2-3B-Instruct         | llama-3b          | 3.21B          | 128K       | 256   | 128       | [Llama-3.2-3B-Instruct](https://huggingface.co/unsloth/Llama-3.2-3B-Instruct) |
| gemma-2-9b-it                 | gemma-9b          | 9.24B          | 8K         | 256   | 128       | [gemma-2-9b-it](https://huggingface.co/unsloth/gemma-2-9b-it)                  |
| Phi-3.5-mini-instruct         | phi-3             | 3.82B          | 128K       | 256   | 128       | [Phi-3.5-mini-instruct](https://huggingface.co/unsloth/Phi-3.5-mini-instruct)  |
| zephyr-sft                    | zephyr-7b         | 7.24B          | 8K         | 16    | 16        | [zephyr-sft](https://huggingface.co/unsloth/zephyr-sft)                       |

---

## 📜 License
This project is licensed under the terms of the Creative Commons Attribution 4.0 International ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)) license.

---

## 🤝 Acknowledgements
We acknowledge and thank the National Library of Medicine for creating and maintaining the Medical Subject Headings ([MeSH](https://www.nlm.nih.gov/mesh/meshhome.html)) vocabulary, which provided the foundational ontology data necessary for constructing the MeSH-Rel-4K dataset.
