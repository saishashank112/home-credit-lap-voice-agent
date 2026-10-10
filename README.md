# Home Credit LAP Qualification Voice Agent

An AI voice agent that calls existing **Home Credit** customers about a pre-approved **Loan Against Property (LAP)** offer of up to **₹75,00,000**, runs a preliminary eligibility check, and hands qualified leads to a senior loan expert.

Built for the **SalesAgents AI: AI Intern (Prompt Engineer)** assignment. The whole agent is driven by one structured system prompt, deployed and tested on **Bolna**.

---

## What the agent does

1. Greets the customer and verifies their identity
2. Checks availability (busy → captures a callback time → ends)
3. Presents the loyalty offer and asks consent to proceed
4. Collects **7 eligibility data points**, remembering anything the customer says out of order
5. Ends the call politely, **with a reason**, as soon as a criterion fails
6. Detects existing-loan / EMI-reduction cases and routes them to a transfer specialist
7. Hands qualified customers to a senior loan expert (only after all 7 answers are collected)

## Eligibility rules

| # | Data point | Eligible | Not eligible |
|---|-----------|----------|--------------|
| 1 | Property type | Residential, Commercial, Industrial | **Agricultural** |
| 2 | Ownership | Sole or Joint | none |
| 3 | Original documents | Available for verification | **Photocopies only / none** |
| 4 | Loan amount | Up to ₹75 lakh | Above ₹75 lakh: cap explained, 75 lakh offered; ends if declined |
| 5 | Occupation + income mode | Salaried or Self-employed, paid via **Bank** | **Cash income** |
| 6 | Market value | Customer's estimate (no minimum) | none |
| 7 | Tenure | 3 to 15 years | **Below 3 or above 15 years** |

## Call flow

```
Greeting → Verify name → Busy? ──yes──► Callback time → End
                           │no
                           ▼
              Customer ID (optional) → Offer + consent
                           ▼
        7-item checklist (always asks the earliest unanswered item)
          │ any clear violation ──► Polite disqualification with reason → End
          │ existing loan / EMI ──► Transfer-specialist message → End
          ▼ all 7 answered and passed
              Senior loan expert handoff → End
```

## Design highlights

- **Per-turn algorithm:** *Extract → Validate → Update memory → Ask the next missing question.* This is what lets the agent capture several facts from one sentence and never re-ask.
- **Priority order for conflicts:** wrong person → busy → transfer → disqualification → loan cap → diversions → next question.
- **Handoff gate:** no handoff until all 7 items are answered, so skipped items are asked again.
- **No invented facts:** interest rates, fees, and thresholds are never made up. Product facts come only from `additional_context_from_rag`, otherwise the agent defers to the senior expert.
- **Natural speech handling:** fillers, self-corrections, hedged answers, Indian number words (lakh, crore, dedh, dhai), and interruptions.
- **Voice-first style:** short turns, one question at a time, numbers spoken in words.
- **Language and gender:** follows `language_to_speak` (English/Hindi) and applies the correct gendered Hindi grammar for `agent_gender`.
- **Safety:** never asks for OTP, PIN, card, Aadhaar or PAN; never reveals loan details to an unverified person; honest about being an AI.

## Prompt variables

`company_name`, `customer_name`, `agent_name`, `agent_gender`, `current_date`, `current_day`, `current_time`, `additional_context_from_rag`, `language_to_speak`, `conversation_history`, `customer_utterance`

## Tech stack

| Purpose | Tool |
|---------|------|
| Voice agent platform | Bolna (speech-to-text, LLM, text-to-speech, call recording) |
| Prompt design | Structured natural-language system prompt |
| Test log | Excel |

## Repository structure

```
.
├── README.md
├── system_prompt.txt              # final system prompt
├── test_cases.xlsx                # scenarios, expected vs actual, pass/fail
├── transcripts.docx               # transcripts for every call
└── audio/
    ├── 01_eligible_customer.mp3
    ├── 02_agricultural_property.mp3
    └── ...
```

> Rename the entries above to match the files you actually upload.

## Test scenarios

| # | Scenario | Expected outcome | Result |
|---|----------|------------------|--------|
| 1 | Fully eligible customer | All 7 collected → handoff | |
| 2 | Agricultural property | Immediate disqualification with reason | |
| 3 | Cash income | Immediate disqualification | |
| 4 | Only photocopies of documents | Immediate disqualification | |
| 5 | Tenure of 2 years | Disqualification | |
| 6 | Tenure of 20 years | Disqualification | |
| 7 | Loan of ₹90 lakh | Cap explained, 75 lakh offered | |
| 8 | Joint ownership | Eligible, continues | |
| 9 | Existing loan on the property | Transfer-specialist message → end | |
| 10 | EMI reduction request | Transfer-specialist message → end | |
| 11 | Multiple facts in one sentence | All captured, only next missing item asked | |
| 12 | Busy customer | Callback time captured → end | |
| 13 | Wrong person on the call | No details shared → end | |
| 14 | Interest-rate question mid-call | No invented rate, returns to checklist | |

Fill the **Result** column from `test_cases.xlsx` after your final runs.

## How to reproduce

1. Create an agent on [Bolna](https://www.bolna.ai).
2. Paste the contents of `system_prompt.txt` into the agent's prompt.
3. Set the welcome message to `Hello, may I speak with {{customer_name}}?`.
4. Set the prompt variables, for example `customer_name = Rahul`, `agent_name = Priya`, `agent_gender = female`, `company_name = Home Credit`, `language_to_speak = English`.
5. Place test calls and play the customer using the scenarios above.

## Assumptions

The assignment leaves a few points open. These are the choices made:

- If a customer declines the ₹75 lakh cap, the call ends politely and cites the cap.
- If a customer has both an existing loan and a disqualifying answer, the existing loan takes priority and goes to the transfer specialist.
- Mixed bank and cash income is treated as eligible only if the main income is received in the bank.
- The customer ID is noted for the loan expert, never validated.
- Unspecified cases (for example a retired customer) are recorded as stated rather than disqualified, since no rule exists.

## Limitations

- Prototype only, tested on a limited set of scenarios.
- Speech recognition errors on the platform can occasionally affect number or category capture.
- No real customer data or CRM is connected; variables are test values.

## Author

**Allampally Saishashank**
