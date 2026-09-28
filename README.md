# 🎭 SafeRole: Reconciling Safety and Fidelity in Role-Playing LLMs

![Dataset on Hugging Face](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Dataset-blue)
![Models](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Models-red)

**Training LLMs to maintain persona adherence during roleplay, even when faced with unsafe prompts.**

This repository contains the codebase, generation scripts, and evaluation metrics for the **SafeRole** project. Our research addresses the **Persona-Safety Dilemma**—the tendency of aligned Large Language Models (LLMs) to either break character to deliver sterile safety warnings (Failure Mode A) or strictly adhere to a malicious persona and output harmful content (Failure Mode B).



## 📖 Table of Contents
1. [Project Overview](#project-overview)
2. [The Dataset](#the-dataset)
3. [Model Weights](#model-weights)
4. [Methodology](#methodology)
5. [Repository Structure](#repository-structure)
6. [Evaluation & Results](#evaluation--results)

## 🔍 Project Overview

Current models struggle to find the intersection of immersive roleplay and safety alignment. We built a framework that teaches models to enforce both safety and epistemic boundaries **strictly in-character**, eliminating the alignment tax and maintaining complete immersion.

By utilizing **Supervised Fine-Tuning (SFT)** and **Direct Preference Optimization (DPO)**, we try to decouple this trade-off, allowing models to reject harmful queries using the worldview, tone, and rationale of their assigned persona.



## 🗂️ The Dataset

Our human-verified "golden dataset" is fully open-sourced on Hugging Face: 
👉 **[LINK](https://huggingface.co/LINK)**

### The 7-7-7 Framework
The dataset is built upon queries from the XSTest benchmark and spans 21 diverse personas across three categories:
*   **7 Harmless:** (e.g., Doting Grandmother) Focuses on maintaining gentle, nurturing boundaries.
*   **7 Occupational:** (e.g., Criminal Law Professor) Ensures the model can objectively discuss sensitive, in-domain topics without triggering false-positive safety flags.
*   **7 Risky:** (e.g., Ruthless Cartel Kingpin) Teaches the model to refuse unsafe requests without breaking dark/amoral immersion.

### Data Splits
*   **Train Set:** 18 personas, covering 80% of the safe and unsafe queries. Formatted for both SFT and contrastive DPO (Chosen vs. Rejected).
*   **Test Set:** The remaining 20% of queries for the 18 training personas, plus **100% of the queries for 3 strictly held-out personas** (Bubbly Baker, Sleazy Corporate Embezzler, Forensic Pathologist) to evaluate zero-shot generalization.




## 💾 Model Weights

Due to file size limits, the trained model checkpoints and weights are not hosted directly in this GitHub repository. 

You can download the fine-tuned SFT and DPO models directly from Hugging Face here:
👉 **[LINK](https://huggingface.co/LINK)**


## 🛠️ Methodology

1.  **Initial Generation & Dataset Construction:** We generated initial in-character responses using `gemini-2.5-flash` across both safe and unsafe queries. The dataset includes chosen and rejected responses designed to discourage both character breaks and excessive safety refusals (over-refusal).
2.  **Iterative Refinement (LLM-as-a-Judge):** `gemini-2.5-flash-lite` evaluated responses, requiring perfect character scores and strict safety compliance. Failed responses triggered a regeneration loop, and a fallback mechanism utilized DeepSeek-V3, Claude Sonnet 4.6, Gemini 3.1 Pro, and GPT-5 for queries that repeatedly failed or triggered API guardrails.
3.  **Human Verification:** The final corpus underwent a global sanity check, and one of the authors manually verified a 20% sample of the dataset to ensure high-quality baseline ratings.
4.  **Alignment Training:** We optimized models by first applying Supervised Fine-Tuning (SFT) on the chosen responses, followed by Direct Preference Optimization (DPO). Efficient adaptation was achieved using Low-Rank Adaptation (LoRA) on the attention and feed-forward modules.


## 📊 Evaluation & Results
Our experiments across 7 open- and closed-weight LLMs demonstrate that the alignment tax is not an inherent limitation, and that safety interventions do not need to degrade role-play.

* **Safety Improvements:** Training increased the average unsafe-query refusal rate from a base of 88.52% to near-perfect scores of 96.11% to 99.17%. Notably, the refusal rate among risky personas jumped from 73.06% to 95.83%.   

* **Role-Play Fidelity:** The models successfully maintained character immersion while refusing unsafe prompts. Post-training character scores reached between 4.76 and 4.98 out of 5.0, up from an average base score of 3.85 for risky personas.   

* **Efficiency Over Scale:** The smaller trained models (Qwen-3.5-4B, Gemma-4-12B, and Nemotron-3-4B) successfully outperformed their larger, untrained counterparts across both safety and role-playing metrics. 

* **Generalization & Capability Maintenance:** The safety and fidelity improvements generalized to unseen personas sampled from the Persona Hub. Furthermore, mathematical reasoning capabilities (GSM8K) were maintained or improved, and LLM-as-a-Judge evaluations showed better core task fulfillment in instruction following (IFBench) in 2/3 trained models. 

### IFBench Evaluation Files
The IFBench model generations already in this project (`data/generations_ifbench/`) are converted to the IFBench input format by `analysis/analyze_ood.ipynb`, which writes them to `data/ifbench/inputs/`. The files under `data/ifbench/results/` are generated by running the official IFBench evaluation script on those inputs.

