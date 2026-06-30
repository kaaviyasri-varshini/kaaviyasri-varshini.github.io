Title: Why Your AI Agent Lies to You (Without Meaning To)
Date: 2026-06-29
Category: AI Agents
Tags: AI-agents, debugging, tracing, OpenTelemetry, observability, LangGraph, evaluation, ai-engineering
Slug: why-your-ai-agent-lies-to-you

## Have You Ever Asked a Friend "Why Did You Do That?" and Gotten a Shrug?

That's basically what it feels like to debug an AI agent system.

You watch it mess up. You check the logs. The logs cheerfully tell you *what* happened — it called a tool, it got a response, it spit out an answer — without ever telling you *why* it went wrong.

That gap between "what happened" and "why it happened" is the whole subject of this post. Solving it requires a different mindset than the one most of us bring from regular software debugging.

## The Problem Isn't That Agents Are Complicated. It's That They Fail Differently.

When people first hit this wall, the instinct is to assume "AI is complex, of course it's hard to debug." But there are four very specific, structural reasons agent systems resist the usual debugging playbook.

**They're not consistent on purpose.** A normal web service is predictable: send the same request twice, get the same answer twice. An AI agent isn't built that way. Because of how language models generate text, the same input can lead down a completely different path on a second run. That's not a glitch — it's baked into the design. Which means the classic debugging move of "just run it again and watch" doesn't reliably work anymore.

**Mistakes hide and travel.** Imagine a relay race where the first runner trips slightly, but you don't notice anything wrong until the third runner crosses the finish line in last place. In multi-agent systems, an early agent can make a small error that doesn't blow up until a much later agent produces a visibly wrong result. The error message points at the last runner. The actual stumble happened three steps earlier.

**You can't see the thinking, only the result.** A normal function either returns the right number or it doesn't. With an agent, the *reasoning* is the important part. If it picked the wrong tool, you want to know why it thought that tool was right. That reasoning never shows up in the final answer. It only exists if someone bothered to capture it along the way.

**Time gets messy when things run in parallel.** If several agents are working at once, their actions get mixed together in real time, even though logically one might have nothing to do with another. A plain timestamp can't tell you whether Agent B reacted to Agent A, or just happened to run at the same moment.

## The Fix: Stop Logging, Start Tracing

Here's the conceptual shift that solves most of this: instead of asking "what happened?" (a log), you ask "why did it happen, in what order, caused by what?" (a trace).

Think of a log as a diary entry — one line per event, no connections drawn between them. A trace is more like a family tree. Each unit of work, called a **span**, knows its start time, end time, and — crucially — its *parent*. So a "the agent thought about this" span sits inside "the agent worked on this task" span, which might contain a "the agent called a tool" span underneath it.

Once you have that tree, debugging becomes almost mechanical: find the task that failed, open up its tree, and look for the first point where something looks off. That point — not the final wrong answer — is where the real problem lives.

## The Plumbing: OpenTelemetry

Under the hood, this tracing is built on **OpenTelemetry** — an industry-standard way of recording "what called what, and for how long." Two things make this a smart choice rather than just a technical detail.

First, it's vendor-neutral. The same instrumentation can feed into different dashboards, so you're not locked into one company's tool.

Second, it plays nicely with everything else in your system. Your agent probably also talks to a database or an external API. Those calls show up in the *same* tree as the agent's reasoning steps — because real systems are never purely "agent." They're agents glued into a much bigger machine.

One small piece of cleverness worth knowing about: in a system with multiple agents talking to each other, each message they pass includes a hidden "trace context" tag. That tag is what allows all the agents' individual trace-trees to merge into one unified tree for the whole task, instead of each agent having its own disconnected diary.

## Watching It Happen: Dashboards That Make Sense of the Mess

Capturing all this data is only useful if a human can actually look at it. That's where tools like **Arize Phoenix** and **Langfuse** come in — dashboards that turn the raw trace-tree into something visual and explorable.

Phoenix is particularly good at showing you the structure of a task at a glance: how many times did the model get called, and where did the time go? It also lets you line up evaluation scores next to the traces they came from, so you can filter straight to "show me every time the agent picked the wrong tool."

Langfuse leans more toward production concerns: tracking how much each task costs in tokens, keeping a history of which version of a prompt was used when, and tying traces back to specific users — handy if one particular user keeps triggering expensive or broken agent runs.

You don't have to pick just one. Many teams use Phoenix while building and testing, then lean on Langfuse once the system is live and they care about cost and trends over time.

## Tracing Tells You What Happened. Evaluation Tells You If It Was Right.

Here's a subtlety worth sitting with: even a perfect trace only shows you events — it doesn't tell you whether those events were *correct*. That's a separate job called evaluation, and it makes sense to check correctness at three different zoom levels:

- **The whole pipeline:** Did the final answer come out right?
- **A single agent:** Did this one agent reason and act sensibly?
- **A single step:** Was this one specific tool call or thought correct?

Why bother with all three? Because checking only the final answer can mislead you in both directions. Sometimes two separate mistakes accidentally cancel out and the final answer looks fine — masking a real problem underneath. Other times, the final answer is wrong, but four out of five agents did their job perfectly and one specific agent is the actual culprit. You won't find that out by only checking the ending.

For grading the trickier, more subjective stuff — like "did this reasoning actually make sense?" — teams often use what's called **LLM-as-evaluator**: a separate AI model whose only job is to read the trace and grade it. The catch is that it has to be a *different* model (or at least a more capable one) than the agent being judged. Otherwise it tends to grade its own homework a little too generously.

## The Cherry on Top: Rewinding Time

Here's maybe the most satisfying trick in the whole toolkit: **deterministic replay**.

Remember how agents aren't consistent — same input, possibly different output? Replay sidesteps that entirely. Since every step's output was already captured in the trace, you can rerun the exact same scenario by reusing those stored outputs instead of asking the model to think it through again.

It's like rewatching a recorded basketball game instead of asking the players to replay it live — you get the exact same sequence of events every time, on demand.

This is incredibly useful for confirming that a bug fix actually worked without rolling the dice on a new, possibly-different model response. You can even do a "partial replay," where the model's past thoughts are reused but a specific tool call is run live — perfect for testing a tool fix in isolation.

## The Takeaway

Debugging AI agents isn't about better logging. It's about capturing the *reasoning trail*, not just the events, and being able to walk back through it like a family tree until you find the first place things went sideways.

Once you can see the "why" and not just the "what" — and you can replay it on demand — an agent stops being a mysterious black box and starts being a system you can actually reason about.

The goal was never to build something impressive. It's to build something you can fix when it breaks.

Now you can.
