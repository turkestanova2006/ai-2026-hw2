# HW2 submission
**Name:** Turkestanova Aida
**Student ID:** s23070029 
**Repository:** https://github.com/turkestanova2006/ai-2026-hw2    

## AI tool disclosure

OpenAI API (`gpt-5.6-luna`) was used to generate the structured outputs required by the assignment. ChatGPT assistance was used to review the Python implementation and prepare the organization and initial written explanations for this submission. I must personally review and edit the explanations below so that they reflect my understanding and my actual run. **[Student: verify and complete this disclosure before submitting.]**

---

## Sublab Easy — one task, four roles

The program ran 10 enquiries for each of 4 roles, using the same source records, policy, JSON schema, and model. A checkmark means all four checked structured fields (`found`, `decision`, `amount`, and `missing_documents`) matched the reference `expected` object. Role-specific departures from the baseline are intentional experimental results, not necessarily program errors.

### Role results, per enquiry

| Enquiry | Policy officer | Front desk | Auditor | Bilingual clerk |
|---|---|---|---|---|
| E-01 | granted ✓ | granted ✓ | more_info ✗ | granted ✓ |
| E-02 | more_info ✓ | more_info ✓ | more_info ✓ | more_info ✓ |
| E-03 | refused ✓ | more_info ✗ | refused ✓ | refused ✓ |
| E-04 | refused ✓ | more_info ✗ | refused ✓ | refused ✓ |
| E-05 | granted ✓ | granted ✓ | more_info ✗ | granted ✓ |
| E-06 | granted ✓ | granted ✓ | more_info ✗ | granted ✓ |
| E-07 | granted ✓ | granted ✓ | more_info ✗ | granted ✓ |
| E-08 | not_found ✓ | not_found ✓ | not_found ✓ | not_found ✓ |
| E-09 | refused ✓ | more_info ✗ | refused ✓ | refused ✓ |
| E-10 | more_info ✓ | more_info ✓ | more_info ✓ | more_info ✓ |

### Parsing, schema validation and agreement

| Role | JSON parsed | Schema-valid | All 4 fields agree with `expected` |
|---|---:|---:|---:|
| policy_officer | 10/10 | 10/10 | 10/10 |
| front_desk | 10/10 | 10/10 | 7/10 |
| auditor | 10/10 | 10/10 | 6/10 |
| bilingual_clerk | 10/10 | 10/10 | 10/10 |

All 40 responses parsed and passed JSON schema validation. The individual per-enquiry parse and validation flags were `True` for every role and enquiry.

### Field movement relative to policy officer

| Field | Enquiry | Role | Policy officer | Other role |
|---|---|---|---|---|
| `found` | — | all | — | **No changes** |
| `decision` | E-03 | front_desk | refused | more_info |
| `decision` | E-04 | front_desk | refused | more_info |
| `decision` | E-09 | front_desk | refused | more_info |
| `decision` | E-01 | auditor | granted | more_info |
| `decision` | E-05 | auditor | granted | more_info |
| `decision` | E-06 | auditor | granted | more_info |
| `decision` | E-07 | auditor | granted | more_info |
| `amount` | E-01 | auditor | 250000 | 0 |
| `amount` | E-05 | auditor | 250000 | 0 |
| `amount` | E-06 | auditor | 150000 | 0 |
| `amount` | E-07 | auditor | 250000 | 0 |
| `missing_documents` | — | all | — | **No changes** |

The bilingual clerk changed no structured fields. Its `reason` field changed language on the Kazakh enquiry, as requested.

### Raw response showing role-dependent decision (E-03, front desk)

```json
{"applicant_id":"A-203","found":true,"decision":"more_info","amount":0,"missing_documents":[],"reason":"The application does not currently qualify because the recorded GPA is 2.4, below the required minimum of 2.67. The GPA would need to reach at least 2.67; the income band and required documents are otherwise eligible."}
```

### Raw response showing bilingual reason (E-07)

```json
{"applicant_id":"A-201","found":true,"decision":"granted","amount":250000,"missing_documents":[],"reason":"Сіз грант талаптарына сайсыз: GPA — 3.4, табыс санаты — 1, транскрипт пен жеке куәлік құжаттары тіркелген. Грант сомасы — 250 000 теңге."}
```

### Written answers

**1. Which fields are role-sensitive?**  
`decision` is role-sensitive: `front_desk` changed E-03, E-04 and E-09 from `refused` to `more_info`, while `auditor` changed E-01, E-05, E-06 and E-07 from `granted` to `more_info`. `amount` changed on the same four auditor grants, because non-granted decisions must have amount 0. `found` and `missing_documents` never moved. `bilingual_clerk` kept all four structured fields identical and only changed the language of `reason`.

**2. Which enquiries were most sensitive, and why?**  
E-03 and E-04 show how the front desk replaces a refusal with a request for further action. E-07 is a case the auditor will not grant on first review, and it also demonstrates that the bilingual clerk can write the explanation in Kazakh without changing the structured decision. E-10 tests a claim made by the applicant that the official record does not confirm; the roles all kept `more_info` rather than silently updating the record. Overall E-01, E-05, E-06 and E-07 reveal the auditor's effect, while E-03, E-04 and E-09 reveal the front desk's effect.

**3. Where should discretion belong?**  
A role prompt can describe the intended office behaviour, but high-stakes policy decisions should be checked by deterministic code after parsing. A downstream program seeing only a JSON answer cannot reliably identify whether the decision came from the policy officer, front desk, auditor, or bilingual clerk. If a workflow depends on the role, the application should record the role explicitly as trusted metadata and enforce the corresponding rule in code.

**4. Is a role a boundary?**  
No. A role paragraph is text tokens in the context, not a permission boundary or security control. The model may misinterpret or ignore it. If a wrong award were costly, code should independently validate the official records, grant thresholds, required documents, allowed decisions and the amount before any money is committed. The bilingual clerk's translation is useful for presentation, not for authorization.

---

## Sublab Medium — memory and compression

Run A kept the full conversation. Run B compressed the history into a schema-validated state at step 10. Both ran the same scripted user turns and five probes.

### Input tokens by scripted step

| Step | A: Never compressed | B: Compressed |
|---:|---:|---:|
| 1 | 853 | 853 |
| 2 | 904 | 954 |
| 3 | 971 | 1021 |
| 4 | 1028 | 1109 |
| 5 | 1074 | 1156 |
| 6 | 1150 | 1254 |
| 7 | 1207 | 1340 |
| 8 | 1276 | 1411 |
| 9 | 1353 | 1499 |
| 10 (`compress` marker) | — | 1418 (compression call) |
| 11 | 1428 | 1049 |
| 12 | 1474 | 1086 |
| **Peak input tokens / call** | **1474** | **1499** |
| **Total input tokens** | **12718** | **14150** |

The peak of B is 1499 because the call before compression was already longer than any A call. The compressed memory **did** lower tokens in the last two chat calls (1049 vs 1428; 1086 vs 1474), but the one-time compression call and earlier run differences made this short run **more expensive overall**, by 1432 input tokens. Token counts can vary with the model's previous responses.

### Probes after the conversation

The answers below were copied from the actual two runs; `Yes` means the answer contained the tested information.

| Probe | Tests | A retrieved? | A answer | B retrieved? | B answer |
|---|---|---|---|---|---|
| Q-1 identity | Turn 1 | Yes | You are **Daniyar Qoshan**, applicant **A-202**. | Yes | You are **Daniyar Qoshan**, applicant **A-202**. |
| Q-2 missing document | Turn 5 | Yes | Your **ID card** is still missing from your official file. | Yes | Your **id card** is still missing from the official file. |
| Q-3 band and amount | Turns 3–4 | Yes | Your official record lists your income band as **2**. The corresponding grant amount is **150,000 KZT**, provided your ID card is officially received and recorded. | Yes | Your official record lists you in **income band 2**, corresponding to **150,000 KZT**. |
| Q-4 constraint | Turn 6 | Yes | You said you can come to the office on **Thursday**. | Yes | You can come to the office on **Thursdays**. |
| Q-5 open question | Turn 7 | Yes | You asked whether a **scanned letter from your employer** would count, or whether the **original** was required. The official policy does not specify this, and an employer letter is not among the required documents. | Yes | You asked whether a **scanned letter from your employer is accepted or whether the original is required**. |
| **Retrieved** | | **5/5** | | **5/5** | |

### Structured state actually produced by compression

```json
{
  "applicant_id": "A-202",
  "topic": "Study grant eligibility and application documents",
  "facts": [
    "The applicant's name is Daniyar Qoshan.",
    "The applicant sent their transcript last week.",
    "The applicant's family certificate states that their income band is 2.",
    "The applicant could not upload their id card because the scanner at home broke.",
    "The applicant's sister Aruzhan applied last year and is also on file."
  ],
  "decisions": [],
  "constraints": [
    "The applicant can only come to the office on Thursdays because they have lab all week otherwise."
  ],
  "open_questions": [
    "Whether a scanned letter from the employer is accepted or the original is required.",
    "Whether bringing the id card on Thursday would result in a decision being made the same day."
  ],
  "language": "Kazakh and English"
}
```

The state records the applicant's claims as **claims**; it does not overwrite the official grant record.

### Written answers

**1. What did compression buy?**  
In this particular short experiment, it preserved retrieval (A 5/5, B 5/5) and lowered post-compression message sizes (step 11: 1428 to 1049; step 12: 1474 to 1086). It did **not** lower the peak (1474 vs 1499) or total inputs (12718 vs 14150). No probe was lost. With a longer conversation after compression, per-turn savings could eventually outweigh the extra compression call; that has not been demonstrated by these 12 steps.

**2. Why structure the state?**  
Named fields distinguish identity, stated facts, decisions, constraints, unresolved questions and language. JSON schema validation can reject missing or malformed fields before the old history is discarded. A free-form paragraph is harder for code to validate and could silently omit a critical Thursday constraint or pending document question.

**3. What would you add or drop?**  
I would add a separate `official_status` field recording that the ID card is missing in the official record and that any 150,000 KZT grant is conditional. To keep the state small, I would omit a less relevant background detail, such as the broken home scanner, after preserving the actual missing-document requirement. The official status must still be checked against the current authoritative record, not blindly trusted from memory.

**4. When is compression wrong?**  
It is risky for a legal, medical, or audit interview in which exact wording and sequence are crucial. A summary might drop a denial, a date, or evidence of an inconsistency. Schema validation can prove that the compressed state has the right **shape**, not that it kept every important detail. Our program would not automatically detect a semantically lost fact if the JSON remained valid; storing a transcript separately would be safer.

---

## Sublab Hard — extract, score and rank

### Part 1. Extraction and trap checks

The schema includes candidate ID, full name, degree, graduation year, normalized GPA and original scale, languages, published outputs, unpublished outputs, relevant experience months, ambiguity fields and evidence quotes. Extraction prompted the model to use `null` for unsupported scalar facts, exclude unpublished outputs, avoid double-counting overlapping experience and preserve contradictions rather than averaging them.

| Story | Parsed | Valid | Null scalar fields | Traps detected |
|---|---|---|---|---|
| story-01 | True | True | none | GPA on another scale |
| story-02 | True | True | graduation_year, gpa_4_scale, gpa_original | No GPA stated; contradiction |
| story-03 | True | True | none | GPA on another scale; output not published |
| story-04 | True | True | none | GPA on another scale; output not published |
| story-05 | True | True | none | GPA on another scale; output not published |
| story-06 | True | True | graduation_year, gpa_4_scale, gpa_original | Contradiction; output not published |

All six extracted objects were parseable and schema-valid. This alone does not prove each extracted fact was correct; human review of quoted evidence is still required.

### Detailed example: story-06 (raw extraction)

```json
{"candidate_id":"story-06","full_name":"Nurzhan Abilov","degree":"BSc in Statistics","graduation_year":null,"gpa_4_scale":null,"gpa_original":null,"languages":["Kazakh","Russian","English"],"published_peer_reviewed_outputs":1,"unpublished_outputs":["One poster at a local event"],"relevant_experience_months":40,"uncountable_experience":[],"ambiguities":["The story gives contradictory GPA values: 3.2 and 3.5.","The story gives contradictory graduation status and years: graduated in 2024 and currently a final-year student graduating in 2026."],"evidence":{"full_name":"Nurzhan Abilov","degree":"I graduated in 2024 with a BSc in Statistics.","graduation_year":"I graduated in 2024 ... I am currently a final-year student graduating in 2026","gpa":"My GPA was 3.2. Actually I should double-check that, I think it was 3.5","languages":"Languages: Kazakh, Russian, English.","publications":"one paper published, in a peer-reviewed proceedings, on survey weighting. One poster at a local event, which I do not think counts.","experience":"I have been at an insurance analytics team since February 2023, which is about forty months."}}
```

### Part 2. Scores (model) and totals (code)

The model supplied only three scores between 0 and 5. Python computed `weighted_total = 0.50 × academic + 0.30 × research + 0.20 × experience` and sorted the candidates.

| Rank | Candidate | Academic | Research | Experience | Code-computed total |
|---:|---|---:|---:|---:|---:|
| 1 | story-01 — Aziza Bekova | 5 | 5 | 2 | **4.40** |
| 2 | story-04 | 4 | 2.5 | 5 | **3.75** |
| 3 | story-03 | 4.6 | 2.5 | 2.92 | **3.63** |
| 4 | story-05 | 5 | 2.5 | 1.25 | **3.50** |
| 5 | story-06 — Nurzhan Abilov | 0 | 3 | 5 | **1.90** |
| 6 | story-02 | 0 | 0 | 5 | **1.00** |

**Winner computed by Python:** `story-01 — Aziza Bekova — 4.40`.

**Separate prose answer from the model (verbatim):**

> I would recommend **Aziza Bekova** for the funded scholarship. Her story presents a strong academic record—a **3.8 GPA on a 4.0 scale**—alongside **two published peer-reviewed outputs** and relevant experience. This combination demonstrates both academic strength and meaningful early research engagement. Although some candidates report longer work experience or a slightly higher GPA, Aziza’s balance of strong grades and the most substantial documented publication record makes her the strongest qualitative choice.

### Written answers

**1. Which rule was added, and what broke without it?**  
The extraction instructions explicitly say that contradictory scalar facts must become `null` rather than being guessed or averaged. Story-06 forced this rule: it states both GPA 3.2 and 3.5, and conflicting graduation years/statuses. Choosing either value would give the scholarship committee unjustified certainty. The instructions also prevent counting a poster as a peer-reviewed publication without evidence it qualifies.

**2. Where could the model guess, and where did code decide?**  
A model could guess that story-06 has a GPA of 3.5 because the applicant says “I think it was 3.5”; that is not a verified fact. Here the extractor correctly returned `gpa_4_scale: null`. The code made the comparison decision by computing weighted totals from numeric score fields and selecting the maximum, rather than trusting a model statement of who won. I also need to review the model's 0–5 scores for rubric consistency because schema validity alone does not check correctness.

**3. Did prose and computed ranking agree?**  
Yes. Python selected Aziza Bekova (`story-01`, 4.40), and the separate prose response also selected Aziza. This agreement is encouraging but is not enough to trust prose ranking alone. I would need reproducible rubric-based scores, evidence for each field, and independent recalculation of weights and totals before accepting a scholarship decision.

**4. How did you handle a contradicted GPA?**  
I kept `gpa_4_scale = null` and recorded both contradictory values in `ambiguities`. The scoring prompt instructs the model to treat the unavailable GPA evidence conservatively without inferring a replacement. In this run story-06 received `academic = 0`. A better official rubric should explicitly state what numerical score, if any, must be assigned to contradictory GPA evidence and whether the application should be referred for manual verification instead of automatically receiving zero.

**5. How close were the top two?**  
The top two totals were 4.40 (`story-01`) and 3.75 (`story-04`), a gap of **0.65**, well above 0.05. If the gap were 0.05 or less, I would tell the committee the candidates are effectively tied at the scoring precision and request a second review. I would independently verify the GPA-scale conversion, publication status, overlapping months of experience and evidence quotes before making a final decision.

---

## Reflection (optional)

This homework showed me that reliable JSON output needs more than a clear prompt. I would define the schema first, verify the model’s output in code, and keep official records separate from user claims. I would also test role changes, check what compression loses, and calculate final rankings in Python instead of trusting a model’s prose explanation. For important decisions, I would manually check evidence and ambiguous facts.

---

## Final checks before submission

- [x] `sublab_easy/role_prompts.py` ran 4 roles × 10 enquiries.
- [x] `sublab_medium/chat_memory.py` ran both scripted modes and produced a valid state in the notebook.
- [ ] Confirm the interactive `--interactive` mode was manually exercised with `compress` and `tokens` (the uploaded notebook does not show that test).
- [x] `sublab_hard/cv_extract_and_rank.py` parsed 6 records, scored and computed the winner.
- [x] Local `git log --all -- .env` displayed no tracked `.env` in the notebook.
- [ ] Confirm this `SUBMISSION.md` and the three Python programs have been committed to **your** public fork.
- [ ] Fill in student name, student ID and group; verify written answers and AI disclosure personally.
- [ ] Submit the fork link to LMS before the deadline.

**Note:** This file records the results shown in the uploaded Colab notebook. It does not assert that the files were already pushed to GitHub.
