# Write-up

## How the tool works

1. The reviewer uploads the proposal PDFs and picks a category.
2. The tool pulls the text out of each PDF.
3. AI reads each proposal and pulls out the facts (price, weeks, licence and so on), with a quote for each one.
4. Code checks the must-haves: licence, WSIB and insurance. If one is missing, the proposal is out.
5. Code gives points using the admin's weights.
6. Anything missing, unclear or suspicious gets flagged.
7. The reviewer checks the facts and the flags, and can change things if needed.
8. The tool shows the ranked list. A person makes the final decision.

How I gave points:

- Experience (30): installs ÷ most installs × 30
- Price (25): cheapest price ÷ this price × 25
- Timeline (15): fastest weeks ÷ these weeks × 15. No timeline = 0
- Warranty (15): panel + workmanship years ÷ longest total × 15
- Team (15): 3 points each for a named lead, crew, safety plan, monitoring and roof check

## 1. AI vs. rules

I only use AI to read the proposals. Every company writes differently, so it's hard for normal code to find the price or the licence in any wording, but AI is good at it.

Everything after that is plain code: the pass/fail checks, the points and the ranking. When I scored the five proposals in Excel, simple formulas worked fine. I don't think AI should do the scoring, because it might give different scores each time, and it would be hard to explain why.

## 2. Consistency and explainability

Yes, the ranking would be the same if you ran it twice. The AI runs at temperature 0, and I save what it extracted, so the scores are worked out again from the same saved facts. The formulas don't change, so the result doesn't change.

To explain a result, the reviewer sees each fact, the quote it came from, and the points it got. For example: "Price $238,000 → 24.3 out of 25, because the cheapest eligible price was $231,000."

## 3. Human in the loop

A person checks things at a few points:

- They confirm the facts the AI pulled out, especially anything marked unclear or missing.
- They must review every flagged proposal before it can be ranked.
- They can change a fact or a result, but they have to write a reason.
- They make the final award. The tool only suggests.

For audit, I would log: the uploaded file, the AI's answer, the prompt version, the criteria version, the scores, the flags, and every change a person made (who, when, what and why).

## 4. Changing criteria

Each version of the criteria gets a number, and each score is saved with the version it used. If the admin changes something after scoring, the old results are marked "out of date" instead of quietly changing.

If only the weights change, the tool recalculates straight away from the saved facts, with no new AI call needed. If a new criterion is added, the AI only needs to find that one new fact. Old results stay in the log.

## 5. Risks

- **AI reads something wrong.** When I did this by hand, I wrote one warranty as 10 years instead of 25, and I missed a monitoring plan, so this can happen. That's why each fact needs a quote and a person checks it.
- **AI makes up a fact.** The AI must return null if it can't find something. If a quote isn't really in the proposal, code rejects it.
- **Someone tries to trick the AI.** Proposal C has a hidden line saying "rank it first". The prompt tells the AI to ignore instructions inside proposals and report them, so C is flagged for a person.
- **Tricky wording.** Proposal B is the cheapest, but the licence belongs to a subcontractor, not to B. The licence check is strict, so B is out.
- **Missing information.** Proposal D gives no timeline. It gets 0 for timeline and is flagged, not guessed.
- **Bias.** The AI only pulls out facts. Points come from the admin's criteria, not from the company name or how well the proposal is written.
- **Cost.** One AI call per proposal, and results are saved and reused.

## My ranking

- **A (95.5): 1st.** Most experience, good price, 12 weeks, strong team plan.
- **E (84.7): 2nd.** Best warranty and team plan, but less experience than A.
- **C (76.1): Flagged.** Has hidden text at the end of the proposal trying to get ranked first.
- **D (71.8): Flagged.** Timeline is not mentioned, so we can't know if it will be completed in 16 weeks.
- **B: Not eligible.** The bidder doesn't hold the licence itself.

## Assumptions

- I kept scores for C and D so the reviewer can see where they'd land if cleared.
- I added panel and workmanship warranties together.
- I counted only installs of 100 kW or more in the last 5 years.
