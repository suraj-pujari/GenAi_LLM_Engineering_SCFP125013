# Lab 6 — Tools & Tool Calling with LLMs

| Field | Details |
|---|---|
| Student | Suraj Pujari |
| Roll Number | SCFP125013 |
| Programme | MCA |
| Repository | GenAI_LLM_Engineering_SCFP125013 |
| Notebook | Activity-4/Lab6_Tools_and_Tool_Calling.ipynb |

## Overview

This lab demonstrates how a large language model can work with Python tools instead of answering only from its internal text generation. The notebook defines local and API-backed tools, binds them to a Groq-hosted chat model, displays the model's requested tool calls, executes the selected tools in Python, and passes results back to the model where the workflow requires a grounded final answer.

The exercises compare several LangChain tool interfaces: the @tool decorator, StructuredTool with Pydantic validation, and a BaseTool subclass with synchronous and asynchronous implementations. Some examples use external weather data; several others deliberately use local demo data so that the workflow can be studied without accessing production systems.

## Learning Objectives

After completing this lab, the student should be able to:

- Define simple LangChain tools with @tool.
- Build StructuredTool tools with Pydantic argument validation.
- Implement BaseTool with both synchronous and asynchronous execution methods.
- Bind one or more tools to a chat model and inspect suggested tool calls.
- Execute tool calls in Python and return ToolMessage results to the model.
- Build chained and multi-tool workflows.
- Distinguish simulated/demo data from external API results.
- Apply basic grounding rules so a final response does not invent unavailable tool data.

## Technology Stack

### Python libraries

- Python and Jupyter Notebook
- LangChain Core
- LangChain Groq integration
- Pydantic
- python-dotenv
- requests
- asyncio support for asynchronous invocation

### External services

- Groq API through ChatGroq, using the model openai/gpt-oss-20b
- Open-Meteo geocoding and forecast APIs for the weather exercise

The order, stock, trip, calendar, and restaurant examples use local simulated datasets. They are not connected to production shipping, inventory, travel, calendar, restaurant, or booking services.

## Installation and Setup

### 1. Open the project

~~~bash
cd "/Users/surajpujari/Data/MIT VUP/Sem 3/GenAI"
~~~

If you created a project virtual environment, activate it before starting VS Code or Jupyter:

~~~bash
source .venv/bin/activate
~~~

This activation step is optional; the notebook itself does not prescribe a particular environment name.

### 2. Install dependencies

The notebook's dependency cell installs the following packages:

~~~bash
python -m pip install -q langchain-core langchain-groq pydantic python-dotenv requests
~~~

Restart the selected Jupyter kernel if package installation requests it.

### 3. Configure the Groq key

Create a local project-root file named .env and add only a placeholder/value in this format:

~~~dotenv
GROQ_API_KEY=your_actual_groq_api_key_here
~~~

The notebook searches the current folder and parent folders for .env, then loads GROQ_API_KEY without printing it. A working key and internet connection are required for Groq model calls.

Security rules:

- Never place a real key in this README, notebook output, screenshots, or source control.
- Do not commit .env.
- The repository .gitignore contains both .env and *.env rules.

### 4. Select the notebook kernel and run

1. Open Activity-4/Lab6_Tools_and_Tool_Calling.ipynb in VS Code.
2. Select the Python Jupyter kernel that has the dependencies installed.
3. Run cells in notebook order, starting with setup.
4. Review the tool-call and tool-result output before relying on any final model response.

## Lab Exercises

### Exercise 1 — Order Status Lookup Tool

Approach: @tool.

The notebook defines lookup_order_status(order_id), which normalizes an order ID and searches a local ORDERS dictionary. The records are demo data, not live shipment tracking.

The sample query requests ORD-4521. The saved tool result identifies it as In transit, with the last update Departed regional sorting center. The final-answer instruction requires the model to use only that tool result and state that an ETA is unavailable when no ETA field exists.

### Exercise 2 — Loan EMI Calculator Tool

Approach: StructuredTool with the EMIArgs Pydantic schema.

Inputs are:

- principal: positive float
- annual_rate: percentage from 0 to 30
- tenure_months: positive integer

The tool calculates monthly EMI using the reducing-balance formula. For a positive monthly rate r and n months, the calculation is:

~~~text
EMI = P × r × (1 + r)^n / ((1 + r)^n − 1)
~~~

For a zero rate, it divides the principal equally across the tenure. The saved example uses ₹500,000 at 9% for 24 months and reports ₹22,842.37 monthly EMI.

### Exercise 3 — Unit Converter Toolkit

Approach: two @tool functions bound together.

- convert_temperature supports Celsius and Fahrenheit in either direction.
- convert_distance supports kilometres and miles in either direction.

The sample question requires only the distance tool. Its saved result converts 100 km to 62.14 miles. The model therefore demonstrates selecting the relevant tool from a bound toolkit instead of calling every available tool.

### Exercise 4 — Weather-Based Outfit Recommendation

Approach: chained @tool workflow using Open-Meteo.

The get_weather tool geocodes a city and requests current temperature, precipitation, weather code, wind speed, and observation time from Open-Meteo. The second model round receives the fetched weather object and proposes arguments for recommend_outfit. The Python code uses the fetched temperature and conditions as authoritative values when invoking the outfit tool.

The saved example records Mumbai weather observed at 2026-10-26T11:30: 35.6 °C, clear sky, 0.0 mm precipitation, and 2.8 km/h wind. Its recommendation was light breathable clothes, sun protection, and water. This is a saved observation at its recorded timestamp, not a continuing claim about current weather. Rerunning the cell requests a new observation.

### Exercise 5 — Async Product Stock Checker

Approach: BaseTool subclass.

ProductStockTool implements:

- _run for synchronous lookup
- _arun for asynchronous lookup

The notebook demonstrates both .invoke() and .ainvoke(), then uses the same tool in a model-bound workflow. Product P1023 is a local demo record with 18 units in stock; it is not a live inventory result.

### Exercise 6 — Expense Splitter Tool

Approach: StructuredTool with the SplitArgs Pydantic schema.

The tool validates a positive total amount and a positive number of people, then calculates the equal share. It is invoked manually and through the model-bound flow with the same inputs. The saved example divides ₹2,400 among 4 people and confirms that both approaches return ₹600 per person.

### Exercise 7 — Trip Cost Estimator

Approach: three independent @tool functions.

- get_flight_cost(origin, destination)
- get_hotel_cost(city, nights)
- get_food_budget(city, days)

The notebook combines their outputs into one trip estimate. In the saved Delhi-to-Goa example, the illustrative values are ₹8,500 for flights, ₹9,000 for three hotel nights, and ₹4,000 for four days of food, for a total of ₹21,500.

All prices are local illustrative estimates. They are not live fares, hotel quotes, or restaurant prices.

### Exercise 8 — Health Metrics Toolkit

Approach: two StructuredTool tools constructed by a HealthToolkit class and returned through get_tools().

- calculate_bmi(weight_kg, height_cm)
- calculate_daily_calories(weight_kg, height_cm, age, activity_level)

Both tools retain Pydantic validation. HealthToolkit creates its BMI and calorie tools as class attributes, and get_tools() returns them as one list for model binding.

BMI is calculated from weight and height and is labelled as a screening measure, not a diagnosis. The calorie calculation is a simplified educational estimate using the implemented formula:

~~~text
10 × weight_kg + 6.25 × height_cm − 5 × age − 5
~~~

The result is then multiplied by the selected activity factor. It is not a sex-neutral midpoint calculation, individualized nutrition guidance, or medical advice.

The saved structured result for the sample input is BMI 22.9 and estimated daily calories of 2,579 kcal for a moderately active person. The source code and structured tool result use the minus-5 formula above. The saved model-written narrative contains a plus-5 transcription error, so the source code and structured tool result are the authoritative record; rerun the cell after strengthening its final-answer grounding if a corrected narrative output is required.

### Exercise 9 — Smart Meeting Scheduler

Approach: three @tool functions.

- check_calendar_availability(person, date)
- get_timezone_offset(city)
- book_meeting(person, date, time)

The first query checks availability only. The second demonstrates sequential availability, timezone, and booking operations. Calendar availability, timezone lookup, and booking are simulated; no real calendar is accessed or modified.

The saved booking example records a simulated meeting with Alex on 2026-10-26 at 14:00 Mumbai time, with offset UTC+05:30.

### Exercise 10 — Restaurant Recommendation Assistant

Approach: four @tool functions bound together.

- find_restaurants(cuisine, city)
- get_restaurant_rating(restaurant_name)
- check_restaurant_open(restaurant_name, day)
- estimate_cost_for_two(restaurant_name)

The notebook searches the local restaurant dataset, then obtains the selected restaurant's rating, demo opening-day status, and illustrative cost for two. The saved example selects Demo Trattoria in Pune: rating 4.3/5, open on Thursday in the demo schedule, and ₹1,800 estimated cost for two.

Restaurant names, ratings, schedules, and prices are simulated demo data. They are not live listings, opening hours, menu prices, or booking information.

## Execution Workflow

The notebook uses several related workflows rather than one identical pattern for all exercises:

1. Load .env configuration and initialize ChatGroq.
2. Define local tools, schemas, or toolkit classes.
3. Bind relevant tools to the model when a model-driven selection is required.
4. Receive and display the model's suggested tool calls.
5. Execute the selected tool calls in Python.
6. Create ToolMessage objects from results where the workflow sends results back to the model.
7. Generate a final, grounded response when the workflow implements a final model round.

Exercise 4 uses an explicit two-round chain because outfit selection depends on retrieved weather. Exercises 7–10 also execute multiple required local tools and combine their results. Exercises 5 and 6 additionally compare direct/manual tool invocation with model-bound use.

## Results and Observations

The current notebook contains saved execution outputs for all ten exercises. Examples include:

| Exercise | Saved example |
|---|---|
| 1 | ORD-4521: In transit; ETA unavailable in supplied demo data |
| 2 | ₹22,842.37 monthly EMI for ₹500,000 at 9% over 24 months |
| 3 | 100 km = 62.14 miles |
| 4 | Mumbai weather observation at 2026-10-26T11:30, followed by an outfit recommendation |
| 5 | P1023 in stock with quantity 18 through both sync and async calls |
| 6 | ₹2,400 split among 4 people = ₹600 each |
| 7 | Illustrative Delhi-to-Goa total = ₹21,500 |
| 8 | BMI 22.9 and estimated daily calories 2,579 kcal |
| 9 | Simulated booking with Alex at 14:00 Mumbai time |
| 10 | Demo Trattoria, Pune: 4.3/5 and ₹1,800 for two |

Saved results demonstrate a prior local execution. They do not guarantee that a later API call, model response, weather result, or model-selected tool call will be identical.

## Important Limitations

- Local order and stock data are demonstrations, not operational services.
- Trip estimates are illustrative and not live prices.
- Calendar availability and bookings are simulated; no real calendar is changed.
- Restaurant data is simulated and does not represent live listings, hours, menus, or reservations.
- Open-Meteo data is external and time-dependent. Treat saved observations as historical records at their printed timestamps.
- Groq model calls require an authorized API key, internet access, and available provider capacity.
- Health outputs are educational approximations, not medical, diagnostic, or individualized nutrition advice.
- A model may still paraphrase tool data incorrectly. Verify important values against the tool output and source logic.

## Troubleshooting

### GROQ_API_KEY is missing

Confirm that a project-root .env file exists, contains the variable name GROQ_API_KEY, and is readable by the selected notebook kernel. Do not print the key to diagnose the issue. Restart the kernel after changing environment configuration.

### Package import fails

Install the exact dependencies listed above into the same Python environment selected by the notebook, then restart the kernel.

### Wrong Jupyter kernel

In VS Code, use the kernel selector in the notebook toolbar and choose the Python environment where LangChain, Pydantic, requests, python-dotenv, and langchain-groq are installed.

### Groq authentication, rate-limit, or network error

Confirm that the key is valid without displaying it, verify internet access, and review the provider error message. Wait and retry later if the provider reports a rate limit. Do not hardcode credentials as a workaround.

### Weather request fails

The weather tool uses timeouts and returns a structured error message when Open-Meteo is unavailable or a city cannot be found. Check connectivity and retry the weather cell; do not substitute invented weather data.

### Notebook requests a restart

Restart the kernel after the dependency-installation cell, then run the notebook from the setup cell onward so imports, tools, and the model client are available in order.

## Repository Structure

Relevant files currently present in the project:

~~~text
GenAI/
├── Activity-4/
│   ├── Lab6_Tools_and_Tool_Calling.ipynb
│   └── Lab6_Tools_and_Tool_Calling_README.md
├── .env                         # Local only; do not commit
├── .gitignore
├── Lab6_Tools_and_Tool_Calling.md
└── README.md
~~~

## GitHub Submission

Review the repository state before staging anything:

~~~bash
git status
git branch --show-current
git remote -v
~~~

Stage only the final Lab 6 notebook and this README:

~~~bash
git add Activity-4/Lab6_Tools_and_Tool_Calling.ipynb \
  Activity-4/Lab6_Tools_and_Tool_Calling_README.md
git diff --cached --check
git diff --cached
~~~

Confirm that the staged diff contains no .env file, API key, backup notebook, or unrelated work. When satisfied, commit and push manually using the repository's existing branch and remote:

~~~bash
git commit -m "Add Lab 6 tools and tool calling submission"
git push
~~~

These commands are documentation only. They are not run automatically by this project.

## Conclusion

Lab 6 demonstrates practical LLM tool calling with local Python functions, validated structured inputs, asynchronous tool execution, chained API-backed reasoning, and multi-tool workflows. It also highlights why tool outputs, source logic, timestamps, and simulated-data labels must be checked before presenting a model response as factual.

## References

- [LangChain Tools documentation](https://docs.langchain.com/oss/python/langchain/tools)
- [LangChain ChatGroq integration](https://docs.langchain.com/oss/python/integrations/chat/groq)
- [Groq Tool Use documentation](https://console.groq.com/docs/tool-use/overview)
- [Groq API reference](https://console.groq.com/docs/api-reference)
- [Pydantic documentation](https://docs.pydantic.dev/latest/)
- [Open-Meteo Weather Forecast API documentation](https://open-meteo.com/en/docs)
