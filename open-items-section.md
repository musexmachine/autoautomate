## Open items (still waiting — plan for them, don't block on them)

The confidence approach is set, but the ranking skill and dashboard approach aren't final yet. Build so each one plugs in later. For each one, start planning now with the steps below, then stop and report before you build it.

### 1. Automatability confidence score (fully local; TypeSafe dropped)

autoautomate is local only. No cloud calls, ever. TypeSafe is hosted only, so it's out. Keep Jev's scoring design, run it on a local model, and let Go own the math and the final rank.

Start planning:
1. Define a `Scorer` interface in Go: `Score(task TaskSummary) (pAutomatable float64, confidence float64, reasons []string, err error)`.
2. **Measure these locally in Go, never send them to a judge as raw data:** frequency per week, total minutes, step variance (edit distance between action sequences), exception rate (the share of runs that leave the usual path), and apps or surfaces touched.
3. **Have the local model rate a short task summary**, with the stats above in view:
   - Noul screen first: "Is this one well-defined, repeatable task?" Drop anything that fails.
   - Score: rules-based vs judgment (4 levels).
   - Score: reversibility (trivial, costly or impossible to undo).
   - Noul: are the inputs structured?
   - Noul: is it a sensitive action (money, credentials, sending as the user)? If yes, force a keep-human prior.
4. **Everything runs locally.** Use a heuristic from the measured stats plus a local LLM (Ollama) answering the rubric above. Label it "uncalibrated". An exception rate over ~30% sets a "redesign first" flag.
6. **Combine with Composite scoring:** weight each dimension in Go, and show the weights to the user.
7. **Rank by expected minutes saved,** which is minutes per week × P(automatable). Show each score's confidence in a column. Rows with low confidence show as "unsure" and aren't ranked.
8. **Never output the "candidate / keep human" label.** The user owns it. Once the user has about 30 labels, use them to calibrate the weights.
9. **Before trusting it,** run 20 to 30 real task summaries through the local scorer and compare them against hand labels. Report the numbers.

### 2. Idea-ranking skill (waiting on skill choice)

Status: after the user labels tasks, autoautomate must rank every "candidate for automation" item as a product or automation idea, giving reasons for each rank. The exact ranking skill isn't picked yet.

Start planning:
1. Search the skill library and trackawesomelist.com for skills or frameworks that rank or vet product ideas (for example ICE/RICE scoring, impact vs. effort, or Dani's `profit-vetting-final-idea` skill for the final pick).
2. Shortlist 2–3. For each, give what it scores, what inputs it needs, and whether it can run fully on a local model.
3. Recommend one with a ⭐ and one sentence on why. Wait for Dani to pick.
4. Output format to design for: a ranked list where each item shows its rank, the score per factor, two or three reasons why it ranks there, and what would move it up.

### 3. Live dashboard approach (research first)

Before you build the dashboard, find skills and libraries for live dashboards in Go. Check the skill library, awesome-go on trackawesomelist.com, and Context7 for current docs. Shortlist 2–3 approaches, for example server-rendered templ + htmx with SSE, or a Go server feeding a small chart library. Star the recommended one and say why. It must run only on localhost.
