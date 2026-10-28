Write-up

How the tool works

Here's the basic flow:





Reviewer uploads the proposal PDFs and picks a category.



Tool extracts the text from each PDF.



AI reads every proposal and pulls out the key facts (price, weeks, licence, etc.) along with a quote for each one.



Code then checks the must-haves: licence, WSIB, and insurance. If any of those are missing, the proposal is out.



Code scores everything using the weights the admin set.



Anything missing, unclear, or suspicious gets flagged.



Reviewer looks over the facts and the flags, and can change things if needed.



Tool shows the ranked list. A person still makes the final call.

How I scored things:





Experience (30): installs ÷ most installs × 30



Price (25): cheapest price ÷ this price × 25



Timeline (15): fastest weeks ÷ these weeks × 15. No timeline = 0



Warranty (15): panel + workmanship years ÷ longest total × 15



Team (15): 3 points each for a named lead, crew, safety plan, monitoring, and roof check

1. AI vs. rules

I only use AI to read the proposals. Companies all write differently, so it's hard for normal code to reliably find the price or the licence no matter how it's worded. AI is actually pretty good at that part.

Everything after that is just regular code — the pass/fail checks, the scoring, and the ranking. When I scored the five proposals in Excel by hand, simple formulas worked fine. I don't think AI should be doing the scoring itself. It could give different numbers each time, and it would be hard to explain why one proposal scored higher than another.

2. Consistency and explainability

Yes, the ranking comes out the same if you run it twice. The AI is set to temperature 0, and I save whatever it extracted. The scores are then calculated again from those saved facts. Since the formulas don't change, the result doesn't change either.

To explain a score, the reviewer can see each fact, the quote it came from, and how many points it got. For example: "Price $238,000 → 24.3 out of 25, because the cheapest eligible price was $231,000."

3. Human in the loop

A person is involved at a few points:





They check the facts the AI pulled out, especially anything marked unclear or missing.



Every flagged proposal has to be reviewed before it can be ranked.



They can change a fact or a score, but they have to write down why.



They make the final award decision. The tool only suggests a ranking.

For the audit trail, I'd log the uploaded file, the AI's response, the prompt version, the criteria version, the scores, the flags, and every change a person made (who changed it, when, what they changed, and why).

4. Changing criteria

Each version of the criteria gets a number, and every score is saved with the version it used. If the admin changes something after scoring has already happened, the old results get marked "out of date" instead of quietly updating.

If only the weights change, the tool can recalculate right away from the saved facts — no need to call the AI again. If a completely new criterion is added, the AI only needs to look for that one new fact. Old results stay in the log either way.

5. Risks





AI reads something wrong. When I went through the proposals by hand, I once wrote a warranty as 10 years instead of 25, and I also missed a monitoring plan. So this can definitely happen. That's why every fact needs a quote, and a person has to check them.



AI makes something up. The AI is told to return null if it can't find something. And if the quote it gives isn't actually in the proposal, the code rejects it.



Someone tries to trick the AI. Proposal C has a hidden line at the end that says "rank it first". The prompt tells the AI to ignore any instructions that appear inside proposals and to report them instead, so C gets flagged for a human to look at.



Tricky wording. Proposal B is the cheapest, but the licence belongs to a subcontractor, not to B itself. The licence check is strict, so B is disqualified.



Missing information. Proposal D doesn't mention a timeline at all. It gets 0 for timeline and is flagged — nothing is guessed.



Bias. The AI only extracts facts. The points come from the admin's criteria, not from the company name or how nicely the proposal is written.



Cost. There's only one AI call per proposal, and the results are saved and reused.

My ranking





A (95.5) — 1st. Most experience, solid price, 12 weeks, strong team plan.



E (84.7) — 2nd. Best warranty and team plan, but less experience than A.



C (76.1) — Flagged. Has hidden text at the end of the proposal trying to get ranked first.



D (71.8) — Flagged. No timeline mentioned, so we can't tell if it would finish within 16 weeks.



B — Not eligible. The bidder doesn't hold the licence themselves.

Assumptions





I kept the scores for C and D so the reviewer can still see where they would land if the flags get cleared.



I added the panel and workmanship warranties together.



I only counted installs of 100 kW or more from the last 5 years.
