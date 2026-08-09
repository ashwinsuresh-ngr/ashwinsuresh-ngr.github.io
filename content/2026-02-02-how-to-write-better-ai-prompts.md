Title: How to Write Better AI Prompts
Date: 2026-02-02
Category: GenAI
Tags: GenAI, LLM, prompt-engineering, best-practices
Slug: how-to-write-better-ai-prompts

This series has covered a lot of ground — tokens, transformers, sampling parameters, chain-of-thought, structured output. But if you just want the practical takeaway — how do you actually write a better prompt, right now, without needing to think about attention mechanisms — this is that post. A distilled, practical guide to getting better results out of any AI model, today.

## Start With the End in Mind

Before writing a single word of your prompt, get clear on three things: What do you actually want the output to look like? Who is it for? How will you use it? A surprising amount of prompt improvement comes just from answering these questions for yourself first, rather than diving straight into typing.

## The Core Habits That Matter Most

- **Be specific, not vague.** "Write about marketing" invites a generic answer. "Write three specific, low-cost marketing ideas for a bakery with a $500/month budget" gives the model something real to work with. Specificity is consistently the single highest-leverage thing you can do to a prompt.
- **Give context the model doesn't already have.** The model doesn't know your audience, your goals, your constraints, or your preferences unless you tell it. A sentence or two of relevant background — who this is for, what's already been tried, what matters most — often improves output more than any clever phrasing trick.
- **Say what you don't want, too.** "Explain this simply" is vaguer than "explain this simply, without jargon, in under 150 words, and skip the historical background." Negative constraints (what to avoid) are just as useful as positive instructions.
- **Show, don't just tell, when format matters.** If you need a very specific structure, tone, or style, one good example is often worth several sentences of description. Don't just describe what you want — demonstrate it.
- **Break big asks into smaller pieces.** A single prompt asking for five different things often does all five poorly. Either number your instructions clearly, or split the task into a sequence of prompts, letting each one build on the last.
- **Ask for reasoning on anything that involves logic.** For math, multi-step problems, or decisions with several factors, simply adding "walk through your reasoning before giving a final answer" measurably improves accuracy — and gives you something to actually check.
- **Specify the format you need.** If you want bullet points, a table, JSON, or a specific length, say so directly. Leaving format unstated means you're leaving that decision up to the model's default guess.

## A Simple Before-and-After

Weak prompt: "Help me with my resume."

Stronger prompt: "I'm a marketing coordinator with 4 years of experience applying for senior marketing manager roles. Review my resume below and suggest 3 specific improvements to make my experience bullet points more results-focused. Keep suggestions concise and actionable."

The difference isn't cleverness — it's information. The stronger prompt tells the model who you are, what you're trying to achieve, what kind of feedback you want, and how much of it.

## A Repeatable Structure for Any Prompt

You don't need to memorize prompt engineering theory to write consistently good prompts. This basic shape covers most situations:

1. **Role or context (optional)** — who should the model act as, or what's the situation?
2. **The task** — what exactly do you want done?
3. **Relevant details** — audience, constraints, background information
4. **Format** — how should the output be structured or how long should it be?

```
[Optional role]: You are a [relevant expertise].
[Task]: [Clearly state what you want.]
[Details]: [Audience, constraints, tone, anything relevant.]
[Format]: [Length, structure, style.]
```

This isn't a rigid template you need to fill out every time — casual questions don't need it — but for anything that matters, running through these four elements mentally (or explicitly) tends to catch the gaps that produce disappointing answers.

## Common Traps Worth Avoiding

- **Assuming the model remembers something you never actually said.** If it's not in the prompt or conversation history, the model doesn't know it.
- **Trusting confident tone as a proxy for accuracy.** Fluent, well-written text can still be wrong — verify specific facts, numbers, and citations independently, especially on unfamiliar topics.
- **Over-explaining simple requests.** Not every prompt needs elaborate structure or reasoning instructions — a quick, clear question is often all a simple task requires.
- **Giving up after one disappointing answer.** Often, a slightly clearer follow-up — adding a missing constraint, pointing out what was off — gets you most of the way to what you actually wanted, faster than starting over.

## Treat It Like a Conversation, Not a Search Query

One of the most underused habits: iterate. If the first response isn't quite right, tell the model specifically what to change — "make it shorter," "more casual," "focus more on X, less on Y" — rather than abandoning the attempt or rewriting the whole prompt from scratch. Modern AI models handle follow-up refinement well, and a short back-and-forth often gets better results faster than trying to perfect a single prompt up front.

## When to Invest More Effort

Not every prompt needs careful engineering. A quick factual question or a casual creative request is usually fine as a simple, direct ask. It's worth investing more deliberate effort — examples, structure, clear constraints — specifically when:

- The task will be repeated often (worth building a reusable template)
- The output needs to match a specific format, tone, or standard
- The task involves multiple steps or reasoning
- Getting it right matters more than getting it fast

## The Bottom Line

Better AI prompts come down to clear, complete communication: say specifically what you want, give the model the context it needs, show examples when format matters, break complex requests into manageable pieces, and iterate rather than settling for a mediocre first answer. None of this requires understanding how transformers work under the hood — it just requires treating the model like what it is: a capable collaborator that can only work with exactly the information and instructions you actually give it.
