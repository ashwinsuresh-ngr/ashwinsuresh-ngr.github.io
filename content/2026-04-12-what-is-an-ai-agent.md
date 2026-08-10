Title: What is an AI Agent?
Date: 2026-04-12
Category: GenAI
Tags: GenAI, AI-agents, LLM, tools, automation
Slug: what-is-an-ai-agent

A chatbot answers your question and stops. An AI agent, by contrast, can decide what to do, take an action, observe the result, and decide what to do next — repeating that cycle until it's actually accomplished a goal. This shift, from generating a single response to autonomously pursuing a task, is what defines an AI agent. Here's what that actually means in practice.

## The Basic Definition

An AI agent is a system, typically built around a large language model, that can perceive its environment, reason about what action to take, execute that action using tools, and use the result to decide its next step — operating in a loop toward a goal, rather than producing one static response to one static prompt.

Where a standard LLM interaction is a single request and a single completion (as covered in the prompt-vs-completion post earlier in this series), an agent introduces a loop: think, act, observe, think again — continuing until the task is done or some stopping condition is reached.

## The Core Components of an Agent

**A reasoning engine.** Almost always an LLM, responsible for interpreting the current state of the task and deciding what to do next — which tool to call, what to search for, whether the task is actually complete.

**Tools.** The mechanisms through which an agent actually acts on the world — searching the web, querying a database, calling an API, executing code, reading or writing files. Without tools, an agent can only "think," never act.

**Memory.** A record of what's happened so far in the task — prior steps taken, information gathered, mistakes made — so the agent's next decision is informed by everything that came before, not made in isolation.

**A control loop.** The logic that ties reasoning, action, and memory together: observe the current state, decide on an action, execute it, observe the outcome, repeat.

## A Simple Walkthrough

Imagine an agent tasked with "find the cheapest flight from New York to Tokyo next month and summarize the top 3 options."

1. The agent reasons: "I need to search flight prices." It calls a flight-search tool.
2. It receives results — a list of flights with prices and times.
3. It reasons again: "I have enough data to identify the cheapest three." It sorts and selects them.
4. It formats a summary and returns it to the user.

A plain LLM asked the same question, without tools, could only guess at plausible-sounding flight prices — and, as covered in the hallucination post earlier in this series, likely fabricate details rather than retrieve real ones. The agent's ability to actually call a tool and act on real, current data is the meaningful difference.

## Agent vs. Chatbot: The Key Distinction

A chatbot, even a very capable one, is fundamentally reactive: it responds to what you say. An agent is goal-directed: given an objective, it determines the steps needed to achieve it, potentially without further human input at each step. This connects to the "AI Agent vs LLM" distinction covered in the next post in this series — an agent is built around an LLM, using it as the reasoning core, but adds the tools, memory, and control loop that let it actually act rather than just respond.

## Why Agents Matter

Agents extend what LLMs can meaningfully do:

- **Access to current, real information** — via tools like search or database queries, rather than relying solely on training data
- **Multi-step task completion** — breaking a complex goal into a sequence of smaller actions, rather than requiring a human to manually orchestrate each step
- **Real-world action** — sending emails, updating records, executing code, rather than only producing text a human has to act on manually

## The Risks That Come With Autonomy

Agentic capability raises the stakes covered in the prompt injection posts earlier in this series. An agent that can take real actions — not just generate text — makes the consequences of a hijacked instruction or a flawed decision meaningfully larger. This is why responsible agent design leans heavily on the safeguards covered in those posts: limited tool permissions, human confirmation for consequential actions, and careful monitoring of agent behavior.

## The Bottom Line

An AI agent is an LLM-powered system built to pursue a goal through a loop of reasoning, tool use, and memory — not just answer a single question, but figure out and execute the steps needed to actually accomplish something. That shift from single-turn response to autonomous, multi-step action is what separates an agent from a chatbot, and it's the foundation for everything from coding assistants to research tools to the kind of multi-agent systems covered later in this series.
