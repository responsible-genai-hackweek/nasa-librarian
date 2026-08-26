# CryoCloud Experiments: LLM + earthdata-mcp

**Date:** 2026-08-26  
**Context:** Testing existing components in a live JupyterHub (2i2c CryoCloud environment)  
**Tools:** Claude Code, JupyterAI, earthdata-mcp (NASA's CMR tool server)

## Experiment Overview

Tested the LLM + earthdata-mcp combination in the CryoCloud JupyterHub using Claude Code and JupyterAI. Explored whether the existing MCP tools could effectively narrow a broad initial question to a focused dataset selection.

### Method

1. Posed a domain question to the LLM
2. LLM queried earthdata-mcp to discover candidate collections
3. Interactively refined the search to filter down to a smaller set closer to the actual need
4. Evaluated the quality of the narrowing process

## Key Finding: The Components Already Exist

**The "NASA librarian" is not a gap that needs new technology — it is a set of existing components that can be stitched together.**

Working alone, none of us would have known about all the recently built components:
- `earthdata-mcp` (NASA, hosted 2026-08, four days old at this hackweek)
- Hosted MCP servers from other teams (SlideRule, datalayer, PODAAC)
- GeoCroissant vocabulary (Rajat Shinde at NASA IMPACT, spec 1.0 published 2026-01-29)
- ChatGSFC (NASA-internal, 9000+ users)
- Element 84's geospatial agent
- SlideRule's schema server (authored/generated/merged pattern)

The hackweek created the **discovery condition** — seeing all these pieces in one room, learning what each team built, understanding how they interlock.

## What Works Today

Claude Code + earthdata-mcp + JupyterAI successfully:
- **Discovered** a broad set of collections relevant to a question
- **Filtered** through interactive refinement to arrive at a smaller, more relevant set
- **Demonstrated** that an LLM can effectively use the MCP to reduce the choice from 287 candidates to ~5 finalists

Experiments were generally favorable. The existing tools do what the "librarian" is supposed to do.

## The Real Gap: Operand, Not Operator

This shifts the project's framing. The MCP and the agent interaction patterns already work. **What is missing is not the agent, but the characterization layer** — the dataset-side facts (resolution, uncertainty, quality flags, cautions) that let the agent *compare* datasets, not just rank them by relevance.

Without those facts:
- An agent can search and filter
- An agent cannot refuse (refuse requires knowing when every candidate fails)
- An agent cannot compare (comparison requires knowing fitness, not just relevance)
- An agent cannot co-register (matching resolutions requires structured resolution data)

**The operand is the record. The operator (agent + MCP) is solved.**

This reframes what the week should deliver: not a new agent, but the metadata layer that makes the existing agents *capable of judgment* rather than just *good at search*.

## Related Notes

- [[notes/2026-08-25-joe-hamman-mcp.md]] — Joe Hamman's demo of MCP and the fitness-blindness problem
- [[notes/2026-08-25-conversations-swinski-hamman.md]] — Conversations anchoring the architecture
- [[decisions.md#d8]] — GeoCroissant vocabulary choice
- [[decisions.md#r2]] — Scope absorption risk: three agents in flight already

## Implication for Scope

This discovery is **risk reduction**: we learned that the integration and plumbing layer (which is hard, slow, and uncertain) is already done. The bottleneck is not agent capability or MCP availability, but data characterization — which is expensive (hand-authored records) but not uncertain.

Betting on the record and the vocabulary (the operand) rather than a new agent (the operator) is now validated.
