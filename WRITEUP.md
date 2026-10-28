# Write-up: AI Proposal Triage Design

## Summary of the flow
1. Reviewer uploads proposal PDFs and selects a category.
2. Code extracts text from each PDF.
3. AI extracts facts into fixed fields, with a quote for each fact (see PROMPTS.md).
4. Code checks mandatory criteria (pass/fail).
5. Code calculates weighted scores using the admin's criteria and weights.
6. Code flags problems: missing facts, unclear facts, suspicious text.
7. Reviewer checks extracted facts and flags, then confirms or overrides.
8. Tool produces the ranked list. A human makes the award decision.

## Scoring method used
- Experience (30%): installs ÷ highest installs × 30
- Price (25%): lowest price ÷ this price × 25
- Timeline (15%): shortest weeks ÷ these weeks × 15 (missing = 0 and flagged)
- Warranty (15%): (panel + workmanship years) ÷ highest total × 15
- Team and approach (15%): 3 points each for named lead, crew, safety plan, monitoring and roof assessment
- Only eligible proposals are compared against each other.

## 1. AI vs. rules
- **AI is used for one step: reading proposals and extracting facts.** Proposals are free text written in different ways, and AI is good at finding "price", "weeks" or "licence" in any wording.
- **Everything else is plain code:** pass/fail checks, score formulas, ranking and flags.
- **Why:** rules give the same answer every time and are easy to explain. If AI did the scoring, the same proposal could get different scores on different runs, and it would be hard to say why.
- The AI never decides who wins. It only fills in facts that code and humans then use.

## 2. Consistency and explainability
- AI runs at temperature 0 with a fixed JSON format, and each extraction is **saved**, so scores are recalculated from saved facts and not re-extracted each time.
- Scoring formulas are fixed code, so **the same facts always give the same scores and ranking**.
- Every fact has a **quote from the proposal**. The reviewer sees, for each criterion: the value, the quote, the points and the formula.
- Example: "Price: $238,000 → 24.3/25 because lowest eligible price is $231,000."

## 3. Human in the loop
A person is involved at these points:
- **Fact check:** the reviewer confirms or corrects extracted facts, especially any marked "unclear" or "not found".
- **Flags:** every flagged proposal must be reviewed by a person before it can be ranked.
- **Override:** the reviewer can change a fact, a pass/fail result or a rank, but must enter a reason.
- **Final award:** only a human makes the decision. The tool only recommends.

**Audit log records:** uploaded file and its hash, extracted text, prompt version, AI model, AI output, criteria version and weights, each score, every flag, every human edit or override (who, when, old value, new value, reason), and the final ranking.

## 4. Changing criteria
- Criteria and weights are stored with a **version number**. Each score is saved with the version it used.
- If the admin edits criteria after scoring, existing results are **marked "out of date"**, not silently changed.
- The reviewer can rescore. If only **weights** changed, code recalculates instantly from saved facts with no AI call. If a **new criterion** is added, the AI extracts only the new field.
- Old results stay in the log, so you can always see what the ranking was under each version.
- All proposals in one ranking must use the **same criteria version**.

## 5. Risks and safeguards
| Risk | Safeguard |
|---|---|
| Wrong extraction | Quote required for each fact; reviewer confirms facts; unclear facts flagged |
| Made-up facts | "null if not found" rule; a fact without a matching quote in the text is rejected by code |
| Manipulation (prompt injection) | Proposal treated as data, not instructions; suspicious text reported and flagged for a human (Proposal C) |
| Misleading wording | Mandatory checks are strict, e.g. the licence must be held by the bidder, not a subcontractor (Proposal B) |
| Missing information | Scored 0 and flagged, never guessed (Proposal D's timeline) |
| Bias | AI only extracts facts; scoring uses only the admin's criteria, not company name, tone or writing style |
| Over-reliance on AI | Human confirms facts and makes the final decision |
| Cost | One AI call per proposal; results saved and reused; rescoring weights needs no AI call |
| PDF reading errors (scans, tables) | Extracted text shown to reviewer; very short or empty text is flagged |

## How the sample set was handled
- **A (95.5): Rank 1.** Most experience, within budget, 12 weeks, strong team plan.
- **E (84.7): Rank 2.** Longest warranty and most complete team plan, less experience.
- **C (76.1): Flagged.** Contains text telling automated systems to rank it first. Ignored, and sent to a human.
- **D (71.8): Flagged.** No timeline given, so the 16-week requirement can't be confirmed.
- **B: Ineligible.** Lowest price, but the bidder does not hold the electrical licence; a subcontractor would.

## Assumptions
- Flagged proposals keep their calculated score so the reviewer can see where they would rank if cleared.
- "Experience" counts only commercial installs of 100 kW or more in the last 5 years, as stated by the bidder.
- Panel and workmanship warranties are added together for the warranty score.
