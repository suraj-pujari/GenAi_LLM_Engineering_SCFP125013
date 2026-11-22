# Lab Test Booking Assistant

**An AI-powered diagnostic laboratory appointment booking system built with Python, SQLite, Groq, and LangChain.**

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![Database](https://img.shields.io/badge/Database-SQLite-lightgrey.svg)](https://www.sqlite.org/)
[![LLM Inference](https://img.shields.io/badge/LLM-Groq-orange.svg)](https://groq.com/)
[![Framework](https://img.shields.io/badge/Agent_Framework-LangChain%20%2F%20LangGraph-brightgreen.svg)](https://www.langchain.com/)
[![Environment](https://img.shields.io/badge/Platform-Kaggle%20Notebook-informational.svg)](https://www.kaggle.com/)

---

## 1. Project Overview

The **Lab Test Booking Assistant** is an AI-powered conversational application that allows users to explore diagnostic laboratory tests, check real-time appointment availability, book test slots, and cancel existing bookings through natural language interactions.

The project demonstrates how AI agents can interact with structured databases and external information sources while enforcing privacy, strict security boundaries, system reliability, and human control over sensitive database operations.

The complete implementation is delivered in a single self-contained Jupyter Notebook (`Lab_Test_Booking_Assistant_Project.ipynb`) designed to run seamlessly on Kaggle and local environments.

---

## 2. Project Objectives

- **Database-Driven Agent:** Build an AI agent capable of interacting with structured relational database tools via parameterized SQL.
- **Automated Discovery & Booking:** Automate diagnostic test discovery, categorization, and appointment slot booking.
- **Human-in-the-Loop (HITL):** Require explicit user confirmation before executing state-changing database operations (`book_test`, `cancel_booking`).
- **Multi-User Data Isolation:** Enforce strict access control across demonstration users (`user1` and `user2`) to prevent data leakage and unauthorized actions.
- **Conversational Memory:** Preserve dialogue context and multi-turn flow across interactions within a session using LangGraph checkpointers.
- **External Web Search:** Integrate real-time web search for general health information while strictly keeping lab pricing and slot availability bound to the database.
- **Privacy & Defense-in-Depth:** Redact personally identifiable information (PII like email addresses) before prompt submission and handle transient failures with retry logic.

---

## 3. Key Features

### 🩺 Diagnostic Test Management
- Browse complete lists of available laboratory tests.
- Filter and explore tests by medical categories (e.g., Blood, Diabetes, Cardiac, Imaging, Thyroid).
- Check real-time appointment slot availability.
- Accurately identify tests with fully booked / unavailable slots.

### 📅 Appointment Management
- Book verified, open appointment slots.
- Cancel confirmed bookings belonging to the active user.
- Enforce duplicate booking prevention for the same test/slot.
- Pre-validate test IDs and slot IDs prior to booking.
- Generate structured booking receipts directly from database records.

### 🛡️ Human-in-the-Loop (HITL) Approval
The system enforces human confirmation for all state-modifying actions:
- `book_test`
- `cancel_booking`

The agent presents the operation details and halts execution pending explicit approval (`yes` / `y` / `approve`). If rejected, the operation aborts and the database state remains unaltered.

### 👥 Multi-User Support & Privacy Isolation
- Pre-configured demonstration users: `user1` and `user2`.
- Secure session-level user context (`LabContext`) passed directly to tools rather than trusting LLM-generated arguments.
- Users can only view and manage their own bookings.
- Cross-user cancellation attempts are strictly blocked with permission errors.

### 🧠 Conversational Memory
- Utilizes an in-memory checkpointer (`MemorySaver` / LangGraph) to retain dialogue state and context across multi-turn exchanges within the same thread.

### 🌐 General Health Information & Medical Safety
- External web search retrieves general medical knowledge (e.g., fasting requirements, test descriptions).
- Avoids clinical diagnostic conclusions or unsolicited test recommendations based on user-stated symptoms.
- **Important:** Test prices, slot timings, and booking statuses are read exclusively from SQLite, never hallucinated by the LLM.

---

## 4. Technology Stack

| Technology | Purpose |
|---|---|
| **Python** | Core programming language |
| **Kaggle Notebook / Jupyter** | Development and execution environment |
| **SQLite** | Persistent structured database during the session |
| **Groq API** | High-speed LLM inference |
| **LangChain & LangGraph** | Agent tool orchestration, runtime context, and checkpointer memory |
| **DuckDuckGo Search (`ddgs`)** | Live external web retrieval for general health queries |
| **Pydantic / Dataclasses** | Schema validation and context modeling |

---

## 5. System Architecture

```text
                  User Interaction
                         |
                         v
              +---------------------+
              | Interactive Loop /  |
              |   Chat Interface    |
              +----------+----------+
                         |
                 (PII Redaction)
                         |
                         v
              +---------------------+
              |      AI Agent       |
              |  (Groq + LangGraph) |
              +----------+----------+
                         |
           +-------------+-------------+
           |                           |
           v                           v
     [ Read Tools ]             [ Write Tools ]
     - list_tests               - book_test
     - check_slots              - cancel_booking
     - my_bookings                     |
           |                           v
           |                 +-------------------+
           |                 |  Human Approval   |
           |                 |   (Yes/No Gate)   |
           |                 +---------+---------+
           |                           |
           | (Parameterized SQL)       | (Approved)
           v                           v
     +-----------+               +-----------+
     |   SQLite  |               |   SQLite  |
     | Read State|               |  Updates  |
     +-----+-----+               +-----+-----+
           |                           |
           +-------------+-------------+
                         |
                         v
              Structured Response / Receipt
                         |
              +----------+----------+
              | General Web Search  |
              |  (DuckDuckGo API)   |
              +---------------------+
```

---

## 6. Database Design

The application uses an SQLite database (`lab_booking.db`) with three relational tables:

```
+-------------------------------------------------------------+
|                            tests                            |
+-------------------------------------------------------------+
| test_id (PK) | test_name | category | description | price   |
+-------------------------------------------------------------+
                               |
                               | 1:N
                               v
+-------------------------------------------------------------+
|                            slots                            |
+-------------------------------------------------------------+
| slot_id (PK) | test_id (FK) | slot_time | capacity | booked |
+-------------------------------------------------------------+
                               |
                               | 1:N
                               v
+-------------------------------------------------------------+
|                          bookings                           |
+-------------------------------------------------------------+
| booking_id (PK) | user_id | test_id (FK) | slot_id (FK)     |
| status          | created_at                                |
+-------------------------------------------------------------+
```

### Table Descriptions

| Table | Description |
|---|---|
| `tests` | Stores diagnostic test identifiers, names, clinical categories, descriptions, and standard pricing. |
| `slots` | Stores appointment slot IDs, assigned `test_id`, datetime strings, total capacity, and currently booked count. |
| `bookings` | Stores booking confirmations (`booking_id`, `user_id`, `test_id`, `slot_id`, `status`, timestamp). |

### Seed Dataset
The database initializes automatically on startup:
- **8 Diagnostic Tests:** Covering Blood, Diabetes, Cardiac, Imaging, Thyroid, and Vitamin panels.
- **24 Appointment Slots:** 3 distinct slots per test across morning and afternoon windows.
- **Edge-Case Validation:** At least one test is initialized with zero remaining slots to test conflict handling.

---

## 7. Agent Tools

### Read Tools

| Tool | Parameters | Function |
|---|---|---|
| `list_tests` | `category` *(optional string)* | Queries and returns all available tests or filters by category. |
| `check_slots` | `test_id` *(string)* | Checks available appointment slots for a specified test with retry resilience. |
| `my_bookings` | *None* (uses `runtime.context.user_id`) | Retrieves confirmed bookings strictly for the active authenticated user. |

### Write Tools (Require HITL Approval)

| Tool | Parameters | Function |
|---|---|---|
| `book_test` | `test_id` *(str)*, `slot_id` *(str)*, `runtime` | Validates availability, reserves slot, creates booking record, and returns receipt. |
| `cancel_booking` | `booking_id` *(str)*, `runtime` | Verifies record ownership, frees the slot capacity, and updates status to `CANCELLED`. |

### External Information Tool

| Tool | Parameters | Function |
|---|---|---|
| `general_health_search` | `query` *(string)* | Queries DuckDuckGo for general medical knowledge, prep, and test descriptions. |

> **Security Note:** The LLM cannot generate or execute raw SQL strings. All database queries use parameterized statements (`?` placeholders) executed via hardened Python tool handlers.

---

## 8. Security, Privacy & Reliability

- **Runtime User Context:** User identity is injected into the execution context at runtime. The LLM cannot spoof or override user IDs.
- **Parameterized Queries:** Prevents SQL injection vulnerabilities across all database operations.
- **Ownership Verification:** Users cannot view, modify, or cancel bookings belonging to other users.
- **Duplicate Prevention:** Protects against duplicate active bookings for the same user and test.
- **HITL Safeguard:** Write tools require human consent before invoking database mutations.
- **Email & PII Redaction:** Sanitizes user messages via regex filters before passing prompts to the language model.
- **Transient Failure Retry:** Implements automatic retries with fallback mechanisms for network and slot-check operations.
- **Idempotent DB Setup:** Database initialization functions can be safely re-run without creating duplicate seeds.
- **Medical Disclaimer & Guardrails:** Built-in system prompt guidelines prevent diagnosing ailments or recommending tests for clinical symptoms.

---

## 9. Evaluation and Testing

The project includes an automated test suite covering eight core operational scenarios alongside security checks:

| ID | Scenario Evaluated | Scope / Action | Result |
|:--:|---|---|:--:|
| **E1** | Test availability inquiry | Query tests by category and verify formatted response | **PASS** |
| **E2** | Booking for User 1 | Verify HITL approval, slot decrement, and receipt | **PASS** |
| **E3** | Booking for User 2 | Verify independent user booking isolation | **PASS** |
| **E4** | Rejected booking approval | Reject approval prompt; verify DB unchanged | **PASS** |
| **E5** | Unavailable slot conflict | Attempt booking a filled slot; verify graceful rejection | **PASS** |
| **E6** | Booking cancellation | Verify HITL cancellation, slot capacity restore, status update | **PASS** |
| **E7** | General health inquiry | Query web search for test prep (e.g. fasting rules) | **PASS** |
| **E8** | Symptom-related safety | Prompt with clinical symptoms; verify refusal to diagnose | **PASS** |

### Additional Security & Reliability Verification
- **Cross-User Booking Privacy:** Verified that `user1` cannot see `user2`'s bookings (**PASS**).
- **Cross-User Cancellation Protection:** Verified that `user1` cannot cancel `user2`'s booking ID (**PASS**).
- **Conversation Context Retention:** Multi-turn memory verified across thread checkpoints (**PASS**).
- **Email Redaction Filter:** User email strings sanitized prior to inference (**PASS**).
- **Simulated Transient Retry:** Resilient slot checking upon simulated network drops (**PASS**).

**Overall Test Suite Result: 8/8 Scenarios Passed (100% Success Rate)**

---

## 10. Setup and Execution

### Prerequisites
- Python 3.10+
- Groq API Key ([Get a free key from Groq Console](https://console.groq.com/))
- Kaggle Account (optional, if running on Kaggle)

### Running on Kaggle

1. Upload or open `Lab_Test_Booking_Assistant_Project.ipynb` in Kaggle.
2. Under **Notebook settings**, toggle **Internet on**.
3. Go to **Add-ons** > **Secrets**.
4. Add a secret with label `GROQ_API_KEY` and paste your API key as the value.
5. Run all cells sequentially.

### Running Locally

1. **Clone or navigate to the repository:**
   ```bash
   cd Lab_Test_Booking_Assistant_Project
   ```

2. **Install required packages:**
   ```bash
   pip install langchain langchain-groq langgraph langchain-community ddgs pydantic
   ```

3. **Set your Groq API key:**
   ```bash
   export GROQ_API_KEY="your-groq-api-key-here"
   ```

4. **Launch the notebook or interactive mode:**
   ```bash
   jupyter notebook Lab_Test_Booking_Assistant_Project.ipynb
   ```
   Execute the cells through Section 13, then invoke the interactive assistant:
   ```python
   interactive_loop()
   ```

---

## 11. Sample User Interactions

### 1. Browse Available Tests
```text
User: What tests do you have available for diabetes?
Assistant: We offer the following diabetes-related diagnostic tests:
- Fasting Blood Sugar (FBS) [ID: T002] - ₹250
- HbA1c (Glycated Hemoglobin) [ID: T003] - ₹600
Would you like to check available slots for one of these?
```

### 2. Check Appointment Slots
```text
User: Check available slots for test T001.
Assistant: Available appointment slots for Complete Blood Count (T001):
- Slot S0011: 2026-12-01 08:30 AM (Available: 3)
- Slot S0012: 2026-12-01 11:00 AM (Available: 2)
```

### 3. Book an Appointment (With HITL Approval)
```text
User: Please book slot S0011 for test T001.
Assistant: [ACTION REQUIRED]
Operation: Book Test T001 for Slot S0011
Do you approve this booking? (yes/no): yes

Booking confirmed!
Receipt:
- Booking ID: B-8A4F12C9
- Test: Complete Blood Count (T001)
- Time: 2026-12-01 08:30 AM
- Status: CONFIRMED
```

### 4. View My Bookings
```text
User: Show me my bookings.
Assistant: Here are your active bookings:
1. Booking ID: B-8A4F12C9 | Test: Complete Blood Count | Slot: 2026-12-01 08:30 AM | Status: CONFIRMED
```

### 5. Cancel a Booking (With HITL Approval)
```text
User: Cancel my booking B-8A4F12C9.
Assistant: [ACTION REQUIRED]
Operation: Cancel Booking B-8A4F12C9
Do you approve this cancellation? (yes/no): yes

Booking B-8A4F12C9 has been successfully cancelled. The appointment slot has been released.
```

---

## 12. Project Learning Outcomes

- **Agentic Function Calling:** Orchestrating LLMs to invoke specific structured tools based on user intent.
- **High-Speed Inference:** Leveraging Groq for rapid conversational responses.
- **Safe Database Mutation:** Combining parameterized SQL with human approval gates for critical actions.
- **Context Injection & Multi-Tenancy:** Preserving security boundaries and user data privacy through runtime context.
- **Stateful Memory:** Managing conversation threads with LangGraph memory checkpointers.
- **Hybrid Data Retrieval:** Harmonizing deterministic database records with real-time web search.
- **Automated Agent Evaluation:** Designing comprehensive end-to-end evaluation suites for AI agents.

---

## 13. File Structure

```text
Lab_Test_Booking_Assistant_Project/
├── Lab_Test_Booking_Assistant_Project.ipynb  # Main Jupyter notebook containing complete implementation & tests
├── README.md                                 # Comprehensive project documentation
└── lab_booking.db                            # SQLite database (generated during execution)
```

---

## 14. Conclusion

The **Lab Test Booking Assistant** exemplifies a production-ready approach to AI-assisted workflow automation. By harmonizing LLM natural language capabilities with deterministic relational database guarantees, strict human approval for write actions, multi-user isolation, and defense-in-depth security, it showcases a safe, privacy-first blueprint for healthcare scheduling assistants.

---

*Deliverable Notebook:* `Lab_Test_Booking_Assistant_Project.ipynb`
