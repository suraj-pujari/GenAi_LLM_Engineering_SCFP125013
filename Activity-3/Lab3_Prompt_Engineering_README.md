Lab 3 - Prompt Engineering

Student: Suraj Pujari
Roll Number: SCFP125013
Course: MCA
Repository: GenAi_LLM_Engineering_SCFP125013

Overview

This lab explores practical prompt engineering techniques using a Large Language Model (LLM) through the Groq API. It demonstrates structured summarization, text classification, information extraction, controlled text generation, translation, and iterative content refinement.

Technologies Used

- Python
- Jupyter Notebook
- Groq API
- python-dotenv
- JSON

Exercises Covered

Exercise Topic Key Work
1 Doctor Summary Generator Generates structured summaries from patient consultation notes using consistent headings.
2 Review Classifier Classifies customer reviews as Positive, Negative, or Mixed.
3 Resume Field Extractor Extracts requested resume fields and returns the results in JSON format.
4 Customer Support Reply Generator Generates polite, professional support replies within a specified word limit.
5 Multi-language Translator Translates text into the requested language while following the specified formality.
6 Iterative Content Refiner Improves a draft through at least three refinement rounds.

How to Run

1. Install Dependencies

Run the following command in the Python environment used by your notebook:

python -m pip install groq python-dotenv

2. Configure the Groq API Key

Create a .env file in the project folder containing the notebook and add:

GROQ_API_KEY=your_groq_api_key

Replace the placeholder with your actual Groq API key.

Security: Never commit the .env file or expose your API key in notebooks, screenshots, or public repositories.

3. Open the Notebook

Open `Activity-3/Lab3_Prompt_Engineering.ipynb` in VS Code or Jupyter Notebook.

4. Run the Cells

Run the notebook cells sequentially, starting with the environment setup and API connection test, followed by Exercises 1–6.

Model Used

The notebook is configured to use the Groq API with the openai/gpt-oss-20b model.

Result

All six exercises were executed successfully in the local Python environment. The notebook demonstrated structured output generation, classification, JSON extraction, word-limited responses, formality-aware translation, and three-round iterative refinement.

Security Note

Keep API credentials private. Ensure .env is excluded from Git tracking before pushing the project to GitHub.
