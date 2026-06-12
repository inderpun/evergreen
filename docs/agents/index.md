# Agents

**LLMs that take actions, not just generate text.**

An agent is an LLM in a loop: it receives a task, decides what to do, calls tools,
observes results, and continues until it reaches a stopping condition. The patterns
in this section are what separate "I connected an LLM to a function" from a system
that is actually reliable.

## What this section covers

| Page | Core idea |
|---|---|
| [Agent patterns](patterns.md) | ReAct, plan-and-execute, reflection, and when to reach for each |
| [MCP (Model Context Protocol)](mcp.md) | The emerging standard for how agents connect to tools and data sources |
| [Multi-agent systems](multi-agent.md) | When to split work across multiple agents and how to coordinate them |
| [Agent memory](memory.md) | What agents remember, how, and where memory tends to break |

## Suggested reading order

Patterns first — every other page assumes you know what an agent loop looks like.
MCP is practical context for anyone building tools. Multi-agent and Memory are
more advanced and can be read in either order.

## After this section you can...

- Design a single-agent loop for a given task and identify its failure modes
- Explain what MCP does and why it matters for tool standardization
- Decide when a multi-agent architecture is justified vs. overcomplicated
- Describe the three types of agent memory and their trade-offs
- Have a clear answer when asked "how do you make agents reliable?"

---

*Section maintained by [Inderpuneet Singh](https://github.com/inderpun) and the Gardener agent.*
