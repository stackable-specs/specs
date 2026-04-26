---
id: fast-agent
layer: interface
extends:
  - python
---

# Fast-Agent

## Purpose

Fast-Agent's value comes from a tight contract between three things: decorator-based agent declarations (`@fast.agent`, `@fast.tool`, `@fast.chain` / `@fast.parallel` / `@fast.router` / `@fast.orchestrator`), framework-managed MCP server connections via the agent's `servers=` list, and out-of-band YAML configuration that pins models and credentials in `fastagent.config.yaml` plus a separate `fastagent.secrets.yaml`. That contract is what makes a Fast-Agent project evaluable, reproducible, and safe to operate. The contract dissolves the moment teams declare agents implicitly at call sites, hardcode provider keys, import MCP server modules directly to "skip the boilerplate", call provider SDKs (`openai`, `anthropic`) around the framework, bolt on a parallel agent framework (LangChain, OpenAI Agents SDK, CrewAI) for "just one workflow", float the `fast-agent-mcp` version, or rely on the function-name default for an agent's `name` and on a global default for `model`. This spec pins the install method, the configuration and secrets layout, the agent-declaration contract, the MCP integration discipline, the workflow composition primitives, and the telemetry posture so a Fast-Agent project remains one coherent program rather than a Python file pointed at whichever model the cloud serves today.

## Do

- Install with `uv tool install fast-agent-mcp` pinned to an exact version, and run agents through `uv run`.
- Declare every agent with `@fast.agent(name=..., instruction=..., model=..., servers=...)` — no implicit name, no implicit model.
- Keep configuration in `fastagent.config.yaml` and credentials in a separate `fastagent.secrets.yaml` that is gitignored.
- Reach MCP tools through the agent's `servers=` list; let the framework own the connection.
- Compose multi-agent flows with `@fast.chain`, `@fast.parallel`, `@fast.router`, or `@fast.orchestrator`.
- Enable OpenTelemetry export so token usage and tool-call traces are inspectable.

## Don't

- Bypass Fast-Agent by calling provider SDKs (`openai`, `anthropic`, `huggingface_hub`) directly from agent code paths.
- Add a parallel agent framework (LangChain, LlamaIndex, OpenAI Agents SDK, AutoGen, CrewAI) to a Fast-Agent project.
- Embed API keys in source, in `fastagent.config.yaml`, or in environment variables baked into container images.
- Import an MCP server's Python module to call its tools instead of declaring it under `servers=`.
- Float the `fast-agent-mcp` version with `^`, `~`, or "latest".
- Rely on the function-name default for an agent's `name` or on a global default for `model` in production code.

## References

- **spec** `python` — Python language baseline this spec extends
- **spec** `uv` — uv toolchain that owns install and execution of fast-agent
- **spec** `opentelemetry` — observability layer fast-agent emits traces into
- **external** `https://fast-agent.ai/` — Fast-Agent documentation home
- **external** `https://fast-agent.ai/agents/defining/` — agent declaration reference
- **external** `https://github.com/evalstate/fast-agent` — Fast-Agent source repository
- **external** `https://modelcontextprotocol.io/` — Model Context Protocol specification

## Rules

1. Install Fast-Agent with `uv tool install fast-agent-mcp@<exact-version>`; do not install with `pip install`, `pipx install`, or an unpinned version specifier. (refs: uv)
2. Pin `fast-agent-mcp` to an exact version in `pyproject.toml` (under `[project.dependencies]` or a `[dependency-groups]` group) and commit `uv.lock`. (refs: uv)
3. Run application scripts and the `fast-agent` CLI through `uv run`; do not invoke them via a system `python` or a hand-managed virtualenv. (refs: uv)
4. Place machine-readable configuration in `fastagent.config.yaml` at the project root.
5. Place provider API keys and other credentials in a separate `fastagent.secrets.yaml`; add it to `.gitignore` and do not commit it.
6. Do not embed credentials in `fastagent.config.yaml`, in Python source, or in environment variables baked into container images at build time.
7. Declare every agent with the `@fast.agent` decorator and pass explicit `name`, `instruction`, `model`, and `servers` arguments; do not rely on the function-name default for `name` and do not omit `model`.
8. Pass an agent's `instruction` as a string literal or a `pathlib.Path` to a markdown file; do not concatenate instructions from runtime values that change per request.
9. Expose MCP server tools to an agent by naming the server in `servers=`; do not import an MCP server's Python module to call its tools directly.
10. Restrict an agent's MCP surface with `tools=`, `resources=`, or `prompts=` when the agent should not see a server's full set.
11. Register a Python function as an agent-scoped tool with `@<agent>.tool` when only one agent should call it; reserve `@fast.tool` for tools intended to be available to every agent.
12. Compose multi-agent workflows with `@fast.chain`, `@fast.parallel`, `@fast.router`, or `@fast.orchestrator`; do not orchestrate agents by hand-calling them in arbitrary Python control flow.
13. Do not call provider SDKs (`openai`, `anthropic`, `huggingface_hub`, `mistralai`, etc.) directly from agent code paths; route every model call through Fast-Agent's model selector.
14. Do not add a parallel agent framework (LangChain, LlamaIndex, OpenAI Agents SDK, AutoGen, CrewAI, Semantic Kernel) as a dependency in a Fast-Agent-managed project.
15. Set `RequestParams(max_iterations=...)` to a value the team has chosen deliberately for production agents; do not rely on the framework default of `99`.
16. Select the model explicitly per agent (`model=` in the decorator) or per invocation (`--model` on the CLI); do not depend on a global default for production code paths.
17. Configure OpenTelemetry export in `fastagent.config.yaml` so the framework's built-in tracing for token usage and tool calls is captured. (refs: opentelemetry)
