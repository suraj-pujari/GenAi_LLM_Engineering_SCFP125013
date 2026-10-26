# Lab 5 — Workflow Design with LLMs

## Prompt Templates and Output Parsers

| Field | Details |
|---|---|
| Student | Suraj Pujari |
| Roll Number | SCFP125013 |
| Programme | MCA |
| Semester | 3 |
| Repository | GenAI_LLM_Engineering_SCFP125013 |
| Notebook | Activity-4/Lab5_Workflow_Design_with_LLMs.ipynb |

## Overview

This lab designs six reusable LLM workflows with LangChain prompt templates and output parsers. Each workflow converts a raw input into structured information, free-form text, or both. Where a workflow has multiple steps, the parsed result from one step is serialized and supplied to the next prompt instead of repeatedly passing the original raw text.

The notebook uses the Groq-hosted model openai/gpt-oss-20b through ChatGroq. It combines PromptTemplate and ChatPromptTemplate with PydanticOutputParser for validated structured data and StrOutputParser for final free-form text.

## Learning Objectives

- Design reusable prompt templates with named input variables.
- Use PromptTemplate for factual extraction tasks.
- Use ChatPromptTemplate with system instructions for generation, evaluation, and reporting tasks.
- Parse structured model responses with PydanticOutputParser and Pydantic schemas.
- Use StrOutputParser for free-form summaries and reports.
- Pass a parsed workflow result safely into a later workflow step.
- Normalize external API data into a validated structure before using it in a recommendation.
- Recognize that structured output and model-generated text still require review.

## Technology Stack

| Technology | Role in this lab |
|---|---|
| Python and Jupyter Notebook | Notebook runtime and workflow implementation |
| LangChain Core | Prompt templates and output parsers |
| LangChain Groq | ChatGroq model integration |
| Groq API | Hosted model inference for openai/gpt-oss-20b |
| Pydantic | Data schemas and parsed-output validation |
| python-dotenv | Loads local environment configuration |
| requests | Calls Open-Meteo in Exercise 4 |
| Open-Meteo API | Weather/geocoding data for the event recommendation workflow |

## Installation and Setup

### 1. Open the project

~~~bash
cd "/Users/surajpujari/Data/MIT VUP/Sem 3/GenAI"
~~~

If a project-specific Python environment has been created, activate it before opening the notebook:

~~~bash
source .venv/bin/activate
~~~

The notebook does not require a particular environment name; select the Python/Jupyter kernel where the required packages are installed.

### 2. Install the notebook dependencies

The notebook's installation cell uses:

~~~bash
python -m pip install -q langchain-core langchain-groq pydantic python-dotenv requests
~~~

Restart the Jupyter kernel if the installation cell requests it.

### 3. Configure local credentials

Create a local .env file at the project root:

~~~dotenv
GROQ_API_KEY=your_actual_groq_api_key_here
~~~

The notebook searches the current directory and parent directories for .env, then reads GROQ_API_KEY without displaying its value. Groq-backed calls require a valid key and network access.

Security requirements:

- Never place a real API key in this README, notebook source, notebook output, screenshots, or Git commits.
- Never commit .env.
- The repository .gitignore contains .env and *.env entries, so local environment files are ignored by Git.

### 4. Run the notebook

1. Open the notebook in VS Code: `Activity-4/Lab5_Workflow_Design_with_LLMs.ipynb`.
2. Select the correct Python Jupyter kernel.
3. Run cells from top to bottom.
4. If a prompt or schema changes, rerun the dependent workflow cells before treating old output as evidence for the new source.

## Exercises

### Exercise 1 — Customer Review to Support Ticket Pipeline

**Purpose:** Convert a raw customer review into a structured complaint and an internal support-ticket summary.

**Workflow:**

1. PromptTemplate plus PydanticOutputParser extracts complaint, product_or_feature, and sentiment.
2. ChatPromptTemplate plus StrOutputParser writes a support-ticket summary from only the structured extraction.

The extraction instruction explicitly treats a named subscription plan, plan tier, page, or feature as product_or_feature. In the saved example, a double charge on a monthly plan is extracted with Negative sentiment, then turned into a support summary.

### Exercise 2 — Research Paper Abstract to Layperson Summary Pipeline

**Purpose:** Extract core study information and explain it to a non-expert audience.

**Workflow:**

1. PromptTemplate plus PydanticOutputParser extracts research_question, method, and key_finding.
2. ChatPromptTemplate plus StrOutputParser produces a plain-language summary from the structured extraction.

The saved example concerns spaced practice and vocabulary retention among adult learners. The structured result retains the stated study limitation that one course does not establish the finding across all ages or languages.

### Exercise 3 — Interview Transcript to Candidate Scorecard Pipeline

**Purpose:** Turn an interview transcript into a structured, evidence-based scorecard.

**Workflow:**

1. PromptTemplate plus PydanticOutputParser extracts question-and-answer pairs.
2. ChatPromptTemplate plus PydanticOutputParser evaluates requested skills with evidence or gaps.
3. ChatPromptTemplate plus PydanticOutputParser produces a structured scorecard and cautious recommendation.

The saved output records evidence for Java and Spring Boot plus API testing, and records a gap for SQL performance tuning. It recommends an additional focused assessment rather than making an automatic hiring decision.

This is a structured evidence-review exercise only. It is not a substitute for fair, human-led hiring practice.

### Exercise 4 — Outdoor Event Plan to Weather-Adjusted Recommendation Pipeline

**Purpose:** Combine structured event details and weather-service data to create a practical organizer recommendation.

**Workflow:**

1. PromptTemplate plus PydanticOutputParser extracts event_type, event_date, location, and indoor_outdoor_preference.
2. PromptTemplate formats the weather query. The notebook calls Open-Meteo, then parses normalized temperature, precipitation chance, and conditions through PydanticOutputParser.
3. ChatPromptTemplate plus StrOutputParser produces an organizer recommendation using the structured event and weather objects.

The saved example is a flexible outdoor community book fair in Pune on 2026-10-25. Its saved weather values are 33.3 °C, 6% precipitation chance, and overcast conditions. Because the date is historical relative to a later run, these values are a saved API observation, not a current-weather claim.

The notebook handles request, location, and missing-data failures by reporting that the weather pipeline could not complete rather than inventing a forecast.

### Exercise 5 — Customer Support Chat Log to Escalation Report Pipeline

**Purpose:** Normalize a support conversation, identify escalation gaps, and create a prioritized handoff report.

**Workflow:**

1. PromptTemplate plus PydanticOutputParser extracts issue, customer_tone, and resolution_attempted.
2. ChatPromptTemplate plus PydanticOutputParser identifies missing_fields and is_ready_to_escalate.
3. ChatPromptTemplate plus StrOutputParser generates an escalation report from the structured summary and readiness review.

The saved example identifies a checkout payment failure, a frustrated and urgent tone, and missing account ID, error code, affected service, and timestamp. The report assigns provisional P3 priority because the available information is incomplete.

### Exercise 6 — Workout Goal to Weekly Plan Pipeline

**Purpose:** Convert a beginner fitness goal into a structured weekly plan that respects available days and equipment constraints.

**Workflow:**

1. PromptTemplate plus PydanticOutputParser extracts goal_type, fitness_level, days_available, and equipment_access.
2. ChatPromptTemplate plus PydanticOutputParser produces a WeeklyWorkoutPlan containing validated WorkoutDay entries.

The source validates days_per_week as an integer from 1 to 7. Workout-day durations are constrained to integers from 0 to 180 minutes; rest entries use 0 minutes. The saved example contains three 30-minute workout days and four rest days, matching the supplied availability of three days per week.

This sample is general fitness information, not medical advice. Anyone with an injury or medical concern should consult a qualified professional before beginning a new routine.

## Workflow Architecture

The notebook follows the instructor's structured workflow pattern:

1. Define a Pydantic schema for the expected structured result.
2. Build a PromptTemplate or ChatPromptTemplate with explicit input variables.
3. Add Pydantic format instructions where the model must return structured data.
4. Invoke the Groq-backed model.
5. Parse the response through PydanticOutputParser or StrOutputParser.
6. Serialize parsed Pydantic output with model_dump_json before passing it to a later prompt.
7. Print each step so the structured intermediate result and final result can be reviewed.

The exact pattern differs by exercise:

- Exercises 1, 2, 5, and 6 use structured extraction followed by text or structured generation.
- Exercise 3 uses three structured stages: extraction, skill evaluation, and scorecard.
- Exercise 4 combines LLM extraction, an external API call normalized by Pydantic, and a final text recommendation.

## Results and Observations

The current notebook contains saved results for all six exercises:

| Exercise | Saved observation |
|---|---|
| 1 | The monthly plan is extracted as the affected product/feature; the ticket reports a duplicate charge and refund issue. |
| 2 | The workflow extracts the study question, method, limitation-aware finding, and a layperson summary. |
| 3 | Java/Spring Boot and API testing are demonstrated; SQL performance tuning is not clearly demonstrated. |
| 4 | The saved Pune event result records 33.3 °C, 6% precipitation chance, overcast conditions, and outdoor precautions. |
| 5 | The escalation review identifies missing account ID, error code, affected service, and timestamp; the report is provisional P3. |
| 6 | The plan has seven entries: three 30-minute workout days and four 0-minute rest days. |

These are saved outputs from a prior execution, not promises that later Groq responses or API values will be identical.

## Important Limitations and Review Notes

- Groq responses can vary across calls, even with the same prompt.
- PydanticOutputParser can fail if a model response does not match the expected schema.
- Groq calls require valid credentials, internet access, and provider availability.
- Exercise 4 relies on Open-Meteo availability and date-supported forecast data. Saved weather values are tied to their recorded event date and run.
- All source inputs are fictional or demo text. They must not be treated as real customer, candidate, event, or health records.
- Exercise 3 is not an automated hiring decision system; its scorecard remains an evidence-review demonstration.
- Exercise 6 is educational general fitness content, not medical guidance.
- The saved Exercise 1 output extracts `monthly plan` as the affected product/feature. If its extraction prompt changes later, rerun the affected cells before treating the saved ticket as current evidence.
- Review model-written summaries against the structured output and raw source. For example, the saved Exercise 2 layperson explanation should be checked for faithful comparison wording before being used as a factual research conclusion.

## Troubleshooting

### GROQ_API_KEY is missing

Confirm that the project-root .env file exists and contains the variable name GROQ_API_KEY. Do not print the key while troubleshooting. Restart the kernel after changing the file.

### Package imports fail

Install the exact dependency list shown in the setup section into the selected notebook environment, then restart the kernel.

### Wrong Python or Jupyter kernel

Use VS Code's notebook kernel selector to choose the Python environment containing langchain-core, langchain-groq, Pydantic, python-dotenv, and requests.

### Groq authentication, rate-limit, or network failure

Confirm the key is authorized without revealing it, check internet connectivity, and inspect the provider error message. Wait and retry if the provider reports rate limiting. Do not hardcode a key as a workaround.

### Pydantic or structured-output validation failure

Inspect the preceding model response and schema fields. Ensure the prompt includes the parser's format instructions, correct the schema or prompt only when required, and rerun the affected workflow from its first dependent cell.

### Open-Meteo request failure

Check the event location, date, and network connection. The notebook's Exercise 4 run cell catches request and missing-data errors and reports an unavailable result rather than inventing weather values.

### Kernel restart required

After dependency installation, restart the kernel and run all setup cells before running individual exercises. Later workflow cells depend on imports, the Groq client, helper functions, and parsers defined earlier.

## Project Structure

Relevant files currently present in the project:

~~~text
GenAI/
├── Activity-4/
│   ├── Lab5_Workflow_Design_with_LLMs.ipynb
│   ├── Lab5_Workflow_Design_with_LLMs_README.md
│   └── Lab5_Workflow_Design_with_LLMs_Suraj_Pujari_SCFP125013.before_exercise1_prompt_update.ipynb
├── .env                                        # Local only; do not commit
├── .gitignore
├── Lab5_Workflow_Design_with_LLMs.md
└── README.md
~~~

The local .env file is configuration only and must remain outside Git commits.

## GitHub Submission

Review the current repository state before staging changes:

~~~bash
git status
git branch --show-current
git remote -v
~~~

~~~bash
git add Activity-4/Lab5_Workflow_Design_with_LLMs.ipynb \
  Activity-4/Lab5_Workflow_Design_with_LLMs_README.md
git diff --cached --check
git diff --cached
~~~

Review the staged diff carefully. Do not stage .env, the backup notebook, API keys, or unrelated files. When the staged content is correct, commit and push manually using the existing branch and remote:

~~~bash
git commit -m "Add Lab 5 workflow design submission"
git push
~~~

The commands above are documentation only. This README does not commit, push, rename the repository, or change the remote configuration.

## Conclusion

Lab 5 demonstrates how prompt templates and output parsers support reusable, multi-step LLM applications. The workflows show a practical pattern: extract or normalize information into validated structures, pass those structures forward, and generate summaries, reports, recommendations, or plans with less manual parsing and clearer intermediate outputs.

## References

- [LangChain Core output parsers](https://reference.langchain.com/python/langchain-core/output_parsers)
- [LangChain PydanticOutputParser reference](https://reference.langchain.com/python/langchain-core/output_parsers/pydantic/PydanticOutputParser)
- [LangChain StrOutputParser reference](https://reference.langchain.com/python/langchain-core/output_parsers/string)
- [LangChain ChatGroq integration](https://docs.langchain.com/oss/python/integrations/chat/groq)
- [Groq API reference](https://console.groq.com/docs/api-reference)
- [Pydantic documentation](https://docs.pydantic.dev/latest/)
- [Open-Meteo Weather Forecast API documentation](https://open-meteo.com/en/docs)
