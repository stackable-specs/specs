---
id: langgraph
layer: interface
extends:
  - python
---

# LangGraph

## Purpose

LangGraph turns an LLM application into an explicit state machine: a typed `State` schema, a set of named nodes, declared edges (static, conditional, or `Command`-driven), and a checkpointer that persists every step so the graph can resume, branch, time-travel, or be inspected after the fact. That structure is the entire reason to choose LangGraph over an imperative LangChain script — it is what makes long-running, human-in-the-loop, multi-agent, and partially-failing workflows operable in production. The structure dissolves the moment a project types its state as a bare `dict`, leans on `InMemorySaver` in production, hides control flow inside node bodies that mutate state to "signal" the next node, swallows `interrupt` semantics behind ad-hoc polling, hand-rolls a FastAPI wrapper instead of `langgraph dev` / `langgraph up`, version-skews `langgraph` against `langchain-core`, or stuffs every step into a single mega-node so the graph degrades into a function. This spec pins the install posture, the typed-state contract, the checkpointer choice, the edge / `Command` discipline, the human-in-the-loop posture, the serving and tracing surfaces, and the framework-coexistence rules so a LangGraph project remains an inspectable, resumable graph — not an imperative agent loop in disguise.

## Do

- Install `langgraph`, `langgraph-checkpoint`, and any `langgraph-checkpoint-<store>` packages pinned together, managed by `uv`.
- Type `State` as a `pydantic.BaseModel` or `TypedDict` with declared `Annotated` reducers for any field that accumulates.
- Define the graph by adding nodes and explicit edges (`add_edge`, `add_conditional_edges`) or by returning `Command(goto=..., update=...)` from a node.
- Use a persistent checkpointer in production (`SqliteSaver`, `PostgresSaver`, or a managed equivalent) keyed on a stable `thread_id`.
- Pause for human input with `interrupt(...)` and resume with `Command(resume=...)`; let the checkpointer hold the suspended state.
- Stream node outputs with `.astream(stream_mode=...)`; serve graphs with the `langgraph` CLI (`langgraph dev` for local, `langgraph up` for runtime).
- Trace runs through LangSmith or OpenTelemetry; treat the run trace as the canonical debugging surface.

## Don't

- Type `State` as a bare `dict` or `Any`.
- Use `InMemorySaver` (or no checkpointer at all) in any environment that runs longer than a single test.
- Encode control flow by mutating state in a node and reading it elsewhere; emit `Command(goto=...)` or use a conditional edge.
- Poll graph state for human-in-the-loop; use `interrupt` and `Command(resume=...)`.
- Hand-roll a FastAPI / Flask wrapper around a graph when `langgraph dev` / `langgraph up` already serves it.
- Version-skew `langgraph` against `langchain-core`; upgrade in lockstep.
- Stuff the entire workflow into one mega-node — split into nodes that each have a single responsibility.

## References

- **spec** `python` — language baseline this spec extends
- **spec** `uv` — toolchain that owns install and execution
- **spec** `langchain` — sibling library; LangGraph builds on `langchain-core` runnables
- **spec** `opentelemetry` — observability layer LangGraph runs can export to
- **external** `https://www.langchain.com/langgraph` — LangGraph product page
- **external** `https://langchain-ai.github.io/langgraph/` — LangGraph Python documentation
- **external** `https://langchain-ai.github.io/langgraph/concepts/persistence/` — checkpointer / persistence reference
- **external** `https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/` — human-in-the-loop with `interrupt`
- **external** `https://langchain-ai.github.io/langgraph/cloud/reference/cli/` — `langgraph` CLI reference
- **external** `https://github.com/langchain-ai/langgraph` — LangGraph source repository

## Rules

1. Pin `langgraph`, `langgraph-checkpoint`, and every `langgraph-checkpoint-<store>` package to the same exact version in `pyproject.toml` (no `^` / `~`); commit `uv.lock`. (refs: uv)
2. Pin `langchain-core` to a version compatible with the chosen `langgraph` release and upgrade them together; do not let the two drift between releases. (refs: langchain)
3. Install dependencies with `uv add` / `uv sync --locked` and run scripts and the `langgraph` CLI through `uv run`. (refs: uv)
4. Define the graph state with `pydantic.BaseModel` or `TypedDict`; do not type the state as `dict`, `Any`, or an untyped `MutableMapping`. (refs: python)
5. Annotate accumulating state fields with their reducer (e.g. `Annotated[list[Message], add_messages]`); do not rely on implicit overwrite semantics for fields that must accumulate.
6. Build the graph with `StateGraph(<StateType>)`, declare nodes with `add_node`, and connect them with `add_edge` / `add_conditional_edges`; do not encode control flow by mutating state in a node and reading it from an unrelated node.
7. Express runtime branching with `Command(goto=..., update=...)` returned from a node, or with `add_conditional_edges`; do not branch by raising exceptions or using sentinel state values.
8. Configure a persistent checkpointer (`SqliteSaver`, `PostgresSaver`, `RedisSaver`, or a managed equivalent) for any graph that runs outside a single test process; do not use `InMemorySaver` in dev, staging, or production.
9. Pass a stable `thread_id` (and `checkpoint_id` when resuming) in the run config; do not generate a new `thread_id` per request when conversational continuity is required.
10. Pause for human input with `interrupt(<payload>)` from a node and resume with `Command(resume=<value>)`; do not poll graph state from an external loop to implement human-in-the-loop.
11. Stream node outputs with `.astream(..., stream_mode="updates" | "messages" | "values" | "custom")`; do not block on `.ainvoke` for a user-facing surface where streaming is available.
12. Serve graphs with the `langgraph` CLI (`langgraph dev` locally, `langgraph up` or `langgraph build` for deployment); do not hand-roll a FastAPI / Flask wrapper that re-implements the graph runtime.
13. Declare a single responsibility per node; split a node into multiple nodes when it would otherwise call the LLM more than once or own more than one external side effect.
14. Compose multi-agent topologies as subgraphs (`add_node(<compiled_subgraph>)`) or as a supervisor / router pattern; do not import a non-LangGraph multi-agent orchestrator into a LangGraph project.
15. Enable LangSmith tracing in development with `LANGSMITH_TRACING=true` and `LANGSMITH_API_KEY=<env>`; in production export traces to LangSmith or OpenTelemetry. (refs: opentelemetry)
16. Load `LANGSMITH_API_KEY`, provider API keys, and other secrets from environment variables at runtime; do not commit them to source or to a tracked config file.
17. Do not add a parallel agent framework (AutoGen, CrewAI, OpenAI Agents SDK, Semantic Kernel) as a dependency in a LangGraph-managed project; LangChain may coexist as the runnables and provider abstraction layer. (refs: langchain)
