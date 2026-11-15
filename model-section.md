## Choose the model

Before you wire up any AI calls, help me pick the model. Do this in order and report back. Don't pick for me.

1. **List the jobs the model does.**
   - Group raw events into named tasks.
   - Write a one-line task summary.
   - Suggest "candidate for automation" or "keep human" with a reason.
   - Write the daily and weekly digests.
   Note which jobs need reasoning and which only need short text.

2. **Set the hardware floor.**
   - Target: an 8 GB Apple Silicon Mac (my m1air) running Ollama, plus a typical student laptop (8–16 GB RAM, no GPU) on Windows and Linux.
   - The default local model must run there at a usable speed.

3. **Shortlist 3–4 local models** from the current Ollama library. Prefer small instruct models in the 1–8B range that support JSON or structured output. Use each model's real, current name and size from the Ollama site. Don't guess.

4. **Test on real data.** Run each shortlisted model on the same 20 real task groups from my own database. Score each model on:
   - grouping accuracy, against my own labels
   - valid JSON rate
   - speed in seconds per digest on the m1air
   - RAM used

5. **Recommend.** Return a table of the results. Star (⭐) the recommended local default and give one sentence on why. autoautomate is local only, so there is no cloud model. Make the model a setting in the settings panel, so the user can switch without a rebuild.
