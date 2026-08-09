Title: Common Prompt Engineering Mistakes
Date: 2026-02-01
Category: GenAI
Tags: GenAI, LLM, prompt-engineering, best-practices
Slug: common-prompt-engineering-mistakes

Most disappointing AI outputs aren't the model's fault so much as the prompt's. After covering a whole toolkit of techniques throughout this series — roles, examples, structure, chain-of-thought, sampling controls — it's worth flipping the lens: what are the recurring mistakes that undermine all of that, even when someone knows the techniques exist? Here's a rundown of the most common ones.

## Mistake 1: Being Vague and Assuming the Model Will "Just Know"

The single most common mistake is under-specifying the request and expecting the model to fill in the gaps correctly. "Write something about our product" leaves the model guessing at tone, length, audience, format, and focus — and as covered in the original prompt engineering post, the model isn't reading your mind, it's reading your words. Vague prompts don't get "smart" answers; they get generic, safest-common-denominator answers, because that's the best guess a model can make without more information.

**Fix:** State the audience, goal, format, length, and any constraints explicitly, rather than assuming they're implied.

## Mistake 2: Overloading a Single Prompt With Too Many Tasks

Cramming five different instructions into one dense paragraph — "summarize this, then critique it, then suggest improvements, then rewrite the intro, then give it a title" — often causes the model to partially miss or underweight some of them, especially without clear structure. The model does its best to address everything, but complex, bundled instructions are exactly where things get dropped.

**Fix:** As covered in the structured prompting post, break multi-part requests into a numbered list, or split genuinely complex tasks into separate sequential prompts.

## Mistake 3: Describing a Format Instead of Showing It

Explaining a desired format in prose — "give me a table-like structure with categories and short explanations" — is often far less reliable than just showing an example of that exact structure. Language is ambiguous in ways a concrete example isn't.

**Fix:** Lean on few-shot examples (covered earlier in this series) for any task where precise formatting, tone, or structure genuinely matters, rather than trying to describe it perfectly in words.

## Mistake 4: Treating Every Task Like It Needs Chain-of-Thought

Chain-of-thought prompting is genuinely powerful for math, logic, and multi-step reasoning — but reflexively adding "think step by step" to every prompt, including simple factual or creative requests, just adds unnecessary length and latency without improving anything. Not every task benefits from an explicit reasoning chain.

**Fix:** Reserve chain-of-thought prompting for tasks that actually involve multi-step logic or calculation; skip it for straightforward requests.

## Mistake 5: Ignoring the Difference Between System and User Prompts

In application contexts, a common mistake is cramming everything — persistent rules, role, formatting requirements, and the specific task — into a single undifferentiated prompt, rather than separating persistent configuration (system prompt) from the dynamic, per-request content (user prompt). This makes behavior harder to maintain and, as covered in the developer-focused post, can create real security gaps around user input handling.

**Fix:** Structurally separate what should persist across every interaction from what changes per request — both for maintainability and for reducing prompt injection risk.

## Mistake 6: Assuming a Persona Grants Real Expertise

Telling a model "You are a board-certified cardiologist" changes tone and framing — it does not grant genuine clinical judgment or guarantee medically accurate information, as covered in the role-based prompting post. A common mistake is over-trusting output specifically because it was requested with an authoritative persona, treating the framing as a guarantee of accuracy rather than just a stylistic lever.

**Fix:** Use roles to shape tone and communication style, but verify factual claims independently, especially for high-stakes domains — the persona doesn't change the underlying reliability of the information.

## Mistake 7: Hoping for Structured Output Instead of Enforcing It

Asking nicely for JSON — "please return this as JSON" — without any additional safeguards works most of the time, but "most of the time" isn't good enough for anything feeding into application logic. A missing comma or an unexpected conversational preamble can break a parser in production.

**Fix:** As covered in the JSON output post, use dedicated JSON mode, schema-constrained generation, or function calling for anything that needs to be reliably parsed by code — don't rely purely on instruction-following.

## Mistake 8: Not Accounting for Ambiguous or Edge-Case Input

A prompt tested against a handful of clean, well-formed examples can behave unpredictably against messy real-world input — empty fields, unusually long text, unexpected formats, or content the prompt author never considered. This mistake is especially common in production systems built and tested against a narrow set of happy-path examples.

**Fix:** Explicitly test prompts against edge cases and unusual inputs before relying on them broadly, and build fallback handling for cases where generation doesn't go as expected.

## Mistake 9: Confusing "It Sounds Confident" With "It's Correct"

A subtler mistake: trusting an answer more because it's fluently written and confidently phrased. As covered in the hallucination post, fluency and accuracy are not the same thing — a model can generate a completely fabricated statistic or citation with exactly the same confident tone as a correct one, because it has no internal mechanism distinguishing "verified" from "plausible-sounding."

**Fix:** Apply extra scrutiny specifically to concrete facts, numbers, quotes, and citations, regardless of how confident or well-written the surrounding text sounds — and use retrieval-augmented approaches where factual grounding genuinely matters.

## Mistake 10: Never Iterating on a Prompt That "Sort of" Works

A prompt that produces an acceptable result most of the time often gets left alone rather than refined — especially in casual use. But small adjustments (adding a constraint, clarifying an ambiguous instruction, adding one good example) often meaningfully improve consistency, and skipping that iteration leaves real quality on the table.

**Fix:** Treat a "good enough" prompt as a starting point, not a finished product — especially for anything used repeatedly, where small improvements compound in value over many uses.

## Mistake 11: Setting Sampling Parameters Without Thinking About the Task

Using default temperature and top-p settings for every task — or worse, cranking temperature up "for more interesting answers" on a task that actually needs precision — mismatches the sampling behavior to the actual goal. As covered in the temperature and top-p post, factual and structured tasks generally want lower randomness; creative tasks benefit from more.

**Fix:** Deliberately match sampling settings to the nature of the task, rather than leaving them at a one-size-fits-all default.

## Mistake 12: Forgetting That Longer Isn't Always Better

There's a tendency to assume more context, more examples, or more detailed instructions always improve results. In reality, overly long prompts can bury the actually important instructions, increase cost and latency unnecessarily, and sometimes cause the model to underweight content that's less prominent within a very long context.

**Fix:** Include what's genuinely necessary — clear instructions, relevant context, well-chosen examples — but avoid padding a prompt with information that doesn't meaningfully improve the output.

## The Bottom Line

Most prompt engineering mistakes aren't about missing some advanced technique — they're about misapplying or skipping the fundamentals covered throughout this series: being specific rather than vague, structuring complex requests clearly, showing rather than describing, enforcing rather than hoping for structured output, and staying skeptical of confident-sounding but unverified claims. Avoiding these common pitfalls often matters more than mastering any single advanced trick — good prompt engineering is less about clever tricks and more about clear, deliberate communication with a system that can only work with exactly what you give it.
