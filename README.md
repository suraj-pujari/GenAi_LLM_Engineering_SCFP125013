# Generative AI & LLM Engineering

MCA Semester 3 practical work covering large language models, prompt engineering, structured LLM workflows, and tool calling.

| Student | Details |
|---|---|
| Name | Suraj Pujari |
| Roll No. | SCFP125013 |
| Programme | MCA |
| Semester | 3 |

## Overview

This project contains the selected Generative AI lab notebooks and their supporting instructions. The work explores how language models are inspected, tokenized, executed locally and in cloud runtimes, guided with prompts, composed into structured workflows, and connected to Python tools.

The notebooks retain saved outputs from the recorded experiments. They are not a claim that every notebook was executed in one uninterrupted session, and model/API output can vary when cells are run again.

## Learning Objectives

- Compare language-model size, metadata, context limits, and hardware needs.
- Understand tokenization, token IDs, sampling temperature, and conversational context.
- Contrast local inference, Google Colab GPU inference, and hosted API inference.
- Design prompts for extraction, classification, transformation, and refinement.
- Build multi-step workflows with validated structured handoffs.
- Bind LLMs to Python tools while validating inputs and distinguishing live, historical, demo, and simulated data.

## Lab Index

| Lab | Notebook | Supporting documentation |
|---|---|---|
| Lab 1 — Introduction to LLMs | [Open notebook](Activity-3/Lab1_Introduction_to_LLMs.ipynb) | [Instructor brief](Lab1_Introduction_to_LLMs.md) |
| Lab 3 — Prompt Engineering | [Open notebook](Activity-3/Lab3_Prompt_Engineering.ipynb) | [Lab README](Activity-3/Lab3_Prompt_Engineering_README.md) |
| Lab 5 — Workflow Design with LLMs | [Open notebook](Activity-4/Lab5_Workflow_Design_with_LLMs.ipynb) | [Instructor brief](Lab5_Workflow_Design_with_LLMs.md) · [Lab README](Activity-4/Lab5_Workflow_Design_with_LLMs_README.md) |
| Lab 6 — Tools & Tool Calling | [Open notebook](Activity-4/Lab6_Tools_and_Tool_Calling.ipynb) | [Instructor brief](Lab6_Tools_and_Tool_Calling.md) · [Lab README](Activity-4/Lab6_Tools_and_Tool_Calling_README.md) |

## Lab Summaries

### Lab 1 — Introduction to LLMs

Lab 1 contains ten practical activities:

- Comparing small, medium, and large Hugging Face language models and their specifications.
- Tokenizing words, punctuation, and sentences with token IDs.
- Running DistilGPT2 in a personal Google Colab GPU runtime. The saved experiment records CUDA 13.0, an NVIDIA Tesla T4 with 14.56 GB reported memory, and 20/50/100-token generation measurements.
- Inspecting local hardware and estimating weight memory at FP16 and INT4 precisions.
- Running verified local Ollama inference with `qwen2.5:0.5b` on a MacBook Air M4.
- Using hosted inference, comparing local/Colab/API approaches, testing temperature, and building history-aware and role-based chatbots.

The Colab GPU experiment and local Ollama experiment are separate: Colab uses a CUDA-enabled NVIDIA Tesla T4, while Ollama inference runs locally through the Ollama service.

### Lab 3 — Prompt Engineering

Lab 3 uses Groq-hosted inference for six exercises:

1. Doctor-summary generation from supplied consultation notes.
2. Positive/negative/mixed review classification.
3. Resume-field extraction into JSON.
4. Word-limited customer-support reply generation.
5. Formal and informal translation.
6. Three-round iterative content refinement.

All examples are educational demonstrations. Medical summaries are not medical advice.

### Lab 5 — Workflow Design with LLMs

Lab 5 uses `PromptTemplate`, `ChatPromptTemplate`, `PydanticOutputParser`, and `StrOutputParser` to pass structured outputs between steps. Its six workflows cover:

1. Customer review to support ticket.
2. Research abstract to layperson summary.
3. Interview transcript to evidence-based candidate scorecard.
4. Outdoor event details to a weather-adjusted recommendation.
5. Support chat to escalation report.
6. Workout goal to a structured weekly plan.

The weather workflow uses Open-Meteo. Its saved Pune weather result is tied to its recorded event date and is not a current forecast.

### Lab 6 — Tools & Tool Calling

Lab 6 connects a Groq-backed model to validated Python tools. The ten exercises include order status, loan EMI, unit conversion, weather-based clothing, async stock lookup, expense splitting, trip-cost estimation, health metrics, simulated meeting scheduling, and restaurant recommendations.

Open-Meteo is used for the weather exercise. Order, stock, trip, calendar, and restaurant data are local demo or simulated data, not live operational services. Health outputs are educational estimates, not medical advice.

## Environment and Setup

Use a Python Jupyter environment with the dependencies required by the notebook you plan to run. The selected notebooks use these packages:

```bash
python -m pip install huggingface_hub transformers torch psutil requests \
  python-dotenv groq langchain-core langchain-groq pydantic
```

Open a notebook in VS Code or Jupyter, select the Python kernel containing its dependencies, and run cells in notebook order when reproducing an experiment. Some installation cells may request a kernel restart.

For Lab 1 Activity 3, open the Lab 1 notebook in Google Colab and select a GPU runtime before running the CUDA-specific cells. A local MacBook/Apple Metal environment is not a substitute for the Colab NVIDIA CUDA measurement.

For Lab 1 Activity 5, start the local Ollama service and ensure the selected model has been downloaded. The saved demonstration uses `qwen2.5:0.5b`.

## API Key Safety

- Keep credentials in a local `.env` file.
- Never commit `.env` or paste a real API key into a notebook, README, screenshot, or source file.
- Use [.env.example](.env.example) only as a placeholder template.
- Groq-backed activities require an authorized API key and internet access.

## External Services and Limitations

- Hugging Face access may be needed for model metadata, tokenizer/model downloads, or hosted inference features.
- Google Colab GPU availability depends on the selected runtime and service availability.
- Ollama can perform local inference after a model has been downloaded, provided the local service is running. Downloading a model requires network access; optional external tools or integrations may transmit data separately.
- Groq API activities require network access, provider availability, and authorized credentials.
- Open-Meteo data is time-dependent. Saved weather output is a record for its printed timestamp or event date, not a live forecast.
- Model outputs may vary across runs. Important factual or structured results should be checked against the source data and tool output.

## Submission and Learning Notes

The instructor briefs specify the repository naming pattern `generative_ai_<student-name-or-roll-no>`. Before submission, verify that the GitHub repository name, staged files, and remote repository align with that convention.

Only final notebooks and their supporting documentation should be staged. Keep backup notebooks, unrelated experiments, datasets, local environments, and `.env` files out of the submission unless explicitly requested by the instructor.
