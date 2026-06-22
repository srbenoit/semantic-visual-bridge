# Semantic-Visual Bridge
An architecture to help bride the gap between language-centric LLMs and visual-centric humans - specifically, 
to help LLMs generate visuals through semantic negotiation with a deterministic visualization engine, offering
opportunities for human-in-the-loop refinement.

## Purpose & context 
A "Semantic-Visual Bridge" system — an architecture where an LLM expresses semantic intent for visualizations 
while a deterministic pipeline handles translation to geometry and rendered output. The project aims to produce
a fully implementable specification, evolving from an initial design document (v0.1). 

Claude's cross-domain breadth is a genuine comparative advantage to be used actively throughout collaboration. 
Because Claude has no memory between sessions, the documents in this repo are the continuity mechanism — making
their rigor and completeness structurally essential.

### Current state

Three persistent document types have been established as session scaffolding:
- Architecture Spec: the evolving implementable specification
- Decision Log: closed decisions
- Open Questions Registry: open questions with an explicit dependency graph
