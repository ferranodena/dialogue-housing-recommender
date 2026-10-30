# Housing Recommendation Dialogue Systems

**Two approaches to conversational real-estate recommendation: a rule-based dialogue system and an LLM-powered assistant with tools and guardrails.**

![Python](https://img.shields.io/badge/Python-3-3776AB?logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![Transformers](https://img.shields.io/badge/Hugging%20Face-Transformers-FFD21E?logo=huggingface&logoColor=black)
![Model](https://img.shields.io/badge/LLM-Qwen2.5--Coder--3B--Instruct-6E56CF)
![Course](https://img.shields.io/badge/UPC--FIB-TVD-0B5CAD)

This project was developed for **Tractament de la Veu i del Diàleg (TVD)** in the *Grau en Intel·ligència Artificial* at **UPC – FIB**. It explores how the same task, recommending properties from a user's preferences, behaves under two very different dialogue paradigms.

---

## Overview

The system recommends houses from a small real-estate database (`house_data.json`) by collecting preferences such as operation type, location, budget, number of bedrooms, and additional features. Both implementations share the same domain and dataset, which makes their behaviour directly comparable.

| Approach | Language understanding | Control | Main trade-off |
| --- | --- | --- | --- |
| **Rule-based** | Normalisation, regex, and explicit NLU rules | Fully deterministic | Predictable, but limited to what the rules anticipate |
| **LLM + tools** | Qwen2.5-Coder-3B-Instruct with ReAct-style tool calling | Prompt + code guardrails | Flexible natural language, requiring validation around the model |

---

## The two systems

### 1. Rule-based dialogue system

`P1_Regles_Entregable_Final.ipynb`

A classical, deterministic dialogue system organised as a hierarchy of three classes:

- **`DialogueSystem`:** base dialogue flow and property filtering.
- **`ImprovedDialogueSystem`:** broader natural-language understanding: accents, casing, punctuation, number formats (`35k`, `35.000`, `35 mil`), written numbers, ordinals, city aliases (*Barna* → Barcelona), and more.
- **`FlexibleDialogueSystem`:** preference confirmation and correction, progressive constraint relaxation (N → N−1 → N−2), and explanatory responses for partial matches.

The conversation follows a system-driven flow, asks for confirmation before searching, and allows the user to edit a single preference, restart, or cancel. The rental domain rule is also enforced: the maximum rent cannot exceed **35% of the user's monthly income**.

### 2. LLM-powered assistant with tools and guardrails

`P1_LLM_Entregable_Final.ipynb`

The second system delegates language understanding and generation to **Qwen2.5-Coder-3B-Instruct**, which can call the `filter_real_estate` tool through a **ReAct-style** loop. The model proposes a JSON tool call, the code executes it, and the result is injected back into the conversation context.

Because a small model cannot be trusted on its own, several layers of protection surround it:

- **Input guardrails:** regex-based detection of prompt injection, role-token manipulation, and off-topic requests.
- **Tool guardrails:** `**kwargs` capture for unsupported filters, safe numeric parsing, controlled error responses, and a hard limit of **5 properties per search**.
- **Data-provenance checks:** the model may only store a value in memory if the user actually provided it.
- **Output guardrails:** response cleanup and a consistent closing phrase when properties are listed.
- **Few-shot prompting:** examples that stabilise the JSON output format, including for Catalan input.

The result is a hybrid design: the LLM handles language, while explicit code enforces the rules that must not be left to probability.

---

## Repository structure

```text
.
├── P1_Regles_Entregable_Final.ipynb   # Rule-based dialogue system
├── P1_LLM_Entregable_Final.ipynb      # LLM assistant with tools and guardrails
├── TVD_informe_practica1.pdf          # Full project report (in Catalan)
├── house_data.json                    # Shared property database
└── README.md
```

---

## Running the notebooks

The notebooks are designed to run in **Jupyter** or **Google Colab**. A GPU is recommended for the LLM notebook.

```bash
pip install nltk transformers torch accelerate ipython
```

Open the notebook you want to explore and run the cells in order. The NLTK resources required by the rule-based system are downloaded automatically. The LLM notebook downloads the model from the Hugging Face Hub on first execution.

Both notebooks load the property database from `house_data.json` in the working directory, so keep it alongside them.

---

## Key takeaways

- Rules offer **full control and transparent errors**, but every new phrasing requires a new pattern.
- LLMs offer **much broader language coverage**, but their failures are subtle and probabilistic.
- The strongest design is **hybrid**: language is delegated to the model, while critical constraints such as valid cities, result limits, the 35% rental rule, and data provenance remain in code.
- Guardrails are most effective when combined: system prompt, input filtering, tool validation, and output control each catch different failures.

---

## Report

The complete analysis, design decisions, test cases, limitations, and flow diagrams are available in:

**[`TVD_informe_practica1.pdf`](TVD_informe_practica1.pdf)**

---

## Authors

**Berta Aulet** · **Ferran Òdena**  
*Grau en Intel·ligència Artificial : UPC · FIB*
