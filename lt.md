# TDS Learning Trace: Assignment - FastAPI Batch Sentiment Analysis
**Student:** [Your Name] | **Roll No:** [Your ID] | **Date:** 2026-05-04

---

## 📊 Assignment Overview
- **Total Time Invested:** 2.5 hours
- **Overall Difficulty:** 3 (Required some library research and debugging JSON schemas)
- **Primary Discovery Method:** LLM-first, followed by official documentation for validation.

---

## 🧩 Question 11: FastAPI Batch Sentiment Analysis

### 1. The Starting Point
- **Initial Confidence:** 3
- **Pre-existing Knowledge:** I had basic knowledge of Python and REST APIs (like Flask), but I had never used FastAPI or its underlying Pydantic data validation before. I also knew basic rule-based NLP concepts but needed to figure out the fastest way to implement a "happy/sad/neutral" classifier without training a heavy ML model locally.

### 2. The Interaction Loop
| Attempt # | Tool/Resource | Action (Prompt or Search Query) | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini 1.5 Pro | "Write a FastAPI POST endpoint that accepts a JSON array of sentences and returns their sentiment." | Code worked, but it relied on a dummy function that just randomly assigned sentiments. I needed a real rule-based analyzer. |
| 2 | Google Search | "Python simple sentiment analysis library rule based" | Discovered `TextBlob` and `VADER` (via `nltk`). Decided to go with VADER as it handles punctuation and capitalization well. |
| 3 | Gemini Live | "How do I map VADER compound scores to strictly 'happy', 'sad', and 'neutral'?" | Provided a threshold-based mapping function (e.g., compound >= 0.05 is happy, <= -0.05 is sad). |
| 4 | Local Testing (curl) | Ran `curl -X POST ...` with the example JSON payload. | **Friction Point:** I got a `422 Unprocessable Entity` error. FastAPI was rejecting my JSON. |
| 5 | FastAPI Official Docs | Searched "FastAPI List of strings in Request Body" | Realized I needed to define a Pydantic `BaseModel` for the exact input structure `{"sentences": [...]}` rather than just passing `List[str]` to the endpoint function directly. |
| 6 | Local Testing | Fixed the Pydantic models and re-ran the POST request. | The response returned `{"result": [...]}`. I re-read the prompt and realized I had a typo in the key; it strictly required `{"results": [...]}`. Fixed the key. |

### 3. Critical Verification
- **How I validated the output:** I spun up the local server using `uvicorn main:app --reload` and used Postman to send the exact test case provided in the assignment prompt. I verified that the output order exactly matched the input order.
- **AI Hallucination/Error Check:** Initially, the LLM suggested setting the VADER threshold for "happy" at `> 0.0` and "sad" at `< 0.0`. I realized this was technically flawed because a score of `0.01` is highly ambiguous. I manually adjusted the thresholds to `>= 0.05` and `<= -0.05` to create a more realistic "neutral" buffer zone, referencing the official VADER documentation to confirm best practices.

### 4. Technical Synthesis
- **The "Aha!" Moment:** Realizing that FastAPI uses Pydantic models not just for parsing, but for automatic data validation and documentation (Swagger UI). Once I explicitly modeled `SentenceRequest` and `SentenceResponse` classes, the `422 Unprocessable Entity` errors disappeared entirely.
- **Key Takeaway:** Always explicitly define your JSON input/output schemas using Pydantic classes in FastAPI rather than relying on loose dictionaries. Also, closely read the assignment's required JSON keys (`results` vs `result`).

---

## 🧠 Meta-Reflection (Course Feedback)
- **Content Gap:** A quick 5-minute tutorial specifically highlighting how to handle nested JSON objects and arrays using Pydantic models in FastAPI would have saved me about 45 minutes of debugging `422` errors.
- **Learning Verdict:** I likely learned more doing this via trial-and-error and reading the docs than I would have in a traditional lecture. Grappling with the HTTP error codes directly made the concept of data validation stick in a way passive listening wouldn't have.
