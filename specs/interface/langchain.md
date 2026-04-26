---
id: langchain
layer: interface
extends:
  - python
---

# LangChain

## Purpose

LangChain's value lives in a small number of orthogonal abstractions — chat models, prompts, output parsers, retrievers, vector stores — that compose through the LangChain Expression Language (LCEL `|` pipe) into a single typed runnable. Held to that contract, a LangChain application is provider-agnostic, streamable, batchable, traceable, and testable, and the same `Runnable` object can be invoked synchronously, asynchronously, in parallel, or as part of a larger graph. The contract dissolves the moment teams reach for the wrong import paths (`from langchain.chat_models import OpenAI` instead of `from langchain_openai import ChatOpenAI`), pull anything from `langchain.experimental` into a production code path, version-skew the `langchain-core` / `langchain` / `langchain-<provider>` packages, mix sync `.invoke()` and async `.ainvoke()` inside the same request, hand-build prompt strings instead of `ChatPromptTemplate`, parse model output with regex instead of a typed `OutputParser`, or sneak provider SDK calls (`openai.ChatCompletion.create(...)`) past the abstraction "just for one feature". This spec pins the install posture, the package layout, the LCEL-composition discipline, the structured-output and streaming contract, the tracing posture, and the framework-coexistence rules so a LangChain project is one composable runnable a team can trace, swap, and test — not a pile of imperative scripts that happen to import `langchain`.

## Do

- Install `langchain`, `langchain-core`, and every `langchain-<provider>` package pinned to the same exact version, managed by `uv`.
- Build chains by composing `Runnable`s with the LCEL `|` operator; declare the runnable once and reuse it.
- Use the dedicated provider package (`langchain-openai`, `langchain-anthropic`, etc.) for chat models and embeddings.
- Define prompts with `ChatPromptTemplate` / `PromptTemplate` and structured output with `with_structured_output(<PydanticModel>)`.
- Pick one execution mode (sync `.invoke` / async `.ainvoke` / streaming `.astream`) per request path and stick with it.
- Trace runs through LangSmith or OpenTelemetry; treat the trace as the canonical debugging surface.

## Don't

- Import from `langchain.experimental` in production code paths.
- Call provider SDKs (`openai`, `anthropic`, `google.generativeai`) directly from chain code.
- Float `langchain*` versions with `^` / `~`, or upgrade them out-of-lockstep.
- Use deprecated import paths (`from langchain.chat_models import ...`, `LLMChain`, `ConversationChain`).
- Build prompts by Python string formatting / f-strings around model input.
- Parse structured output with regex or `eval`; use a typed `OutputParser` or `with_structured_output`.
- Mix `.invoke()` and `await .ainvoke()` inside the same chain — pick one and let the runnable own concurrency.

## References

- **spec** `python` — language baseline this spec extends
- **spec** `uv` — toolchain that owns install and execution
- **spec** `opentelemetry` — observability layer LangChain runs can export to
- **external** `https://www.langchain.com/` — LangChain product home
- **external** `https://python.langchain.com/docs/introduction/` — LangChain Python documentation
- **external** `https://python.langchain.com/docs/concepts/lcel/` — LangChain Expression Language reference
- **external** `https://python.langchain.com/docs/concepts/structured_outputs/` — structured outputs guide
- **external** `https://docs.smith.langchain.com/` — LangSmith tracing and evaluation
- **external** `https://github.com/langchain-ai/langchain` — LangChain source repository

## Rules

1. Pin `langchain`, `langchain-core`, and every `langchain-<provider>` package to the same exact version in `pyproject.toml` (no `^` / `~`); commit `uv.lock`. (refs: uv)
2. Upgrade the `langchain*` package set in a single coordinated PR; do not let `langchain-core` drift from `langchain` between releases.
3. Install dependencies with `uv add` / `uv sync --locked` and run scripts through `uv run`; do not invoke a system `python` against the project. (refs: uv)
4. Import chat models, embeddings, and other provider primitives from the dedicated provider package (`langchain_openai`, `langchain_anthropic`, etc.); do not import from the deprecated `langchain.chat_models` / `langchain.embeddings` paths.
5. Do not import any symbol from `langchain.experimental` (or `langchain_experimental`) in a code path that ships to production.
6. Do not call provider SDKs (`openai`, `anthropic`, `google.generativeai`, `mistralai`, etc.) directly from chain code; route every model call through the `langchain-<provider>` chat model abstraction.
7. Compose chains with the LCEL `|` operator over `Runnable` objects; do not build new chains using `LLMChain`, `SequentialChain`, `ConversationChain`, or other deprecated `Chain` subclasses.
8. Define prompts with `ChatPromptTemplate`, `PromptTemplate`, or `MessagesPlaceholder`; do not assemble prompts by f-string or `str.format` over arbitrary user input.
9. Request structured output with `chat_model.with_structured_output(<PydanticModel>)` or a typed `OutputParser`; do not parse model output with regex, `eval`, or hand-written JSON extraction.
10. Use Pydantic v2 models (`pydantic.BaseModel`) for structured-output schemas, tool argument schemas, and runnable input/output types. (refs: python)
11. Choose a single execution mode per request path (`.invoke`, `.ainvoke`, `.batch`, `.abatch`, `.stream`, `.astream`) and use it consistently; do not mix sync `.invoke` with `await .ainvoke` within the same chain.
12. Stream responses with `.astream` (or `.stream`) for any user-facing chat surface; do not block on a full `.invoke` when streaming is available.
13. Configure variants and per-call overrides through `Runnable.with_config(configurable=...)` and `RunnableConfig`; do not branch on environment by constructing different runnables at call time.
14. Cache deterministic LLM calls with `set_llm_cache(...)` (e.g. `SQLiteCache`) keyed on prompt + model + parameters; do not memoize calls in ad-hoc Python dicts.
15. Enable LangSmith tracing in development by setting `LANGSMITH_TRACING=true` and `LANGSMITH_API_KEY=<env>`; in production, export traces to LangSmith or OpenTelemetry. (refs: opentelemetry)
16. Load `LANGSMITH_API_KEY`, provider API keys, and other secrets from environment variables at runtime; do not commit them to source or to a tracked config file.
17. Do not add a parallel agent framework (LangGraph for graph control may coexist; AutoGen, CrewAI, OpenAI Agents SDK, Semantic Kernel may not) as a dependency in a LangChain-managed project without an explicit ADR.
