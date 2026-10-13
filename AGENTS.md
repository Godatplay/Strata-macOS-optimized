# AGENTS.md

Strata runs the Qwen3.8-Flash-Next mixture-of-experts model (and its Coder, Swift 1.5 and Unsloth variants) on a
normal PC: one NVIDIA or AMD graphics card plus system RAM, on Windows or Linux. It has a C++/CUDA/HIP engine
(`src/`, `include/`), a Python server with an OpenAI- and Anthropic-compatible API and a web app (`serve/`), and a
one-click installer (`setup.py`, started by `START-HERE.bat` / `setup.sh`).

## Installing Strata for a user

Follow **[docs/AI_SETUP.md](docs/AI_SETUP.md)**: check the PC, pick the model by RAM, run setup non-interactively,
start and verify the server, and connect the user's apps. Never expose the server beyond `127.0.0.1` without
`--api-key`. As an alternative to shell commands, Strata's MCP server ([docs/MCP_SERVER.md](docs/MCP_SERVER.md))
offers the same steps as tools.

## Working on the code

- How the engine works, every measured number, the API and all settings: [docs/DETAILS.md](docs/DETAILS.md) and
  the [paper](docs/paper/Strata-Paper.pdf).
- AMD (HIP) build and validation: [docs/AMD_HIP.md](docs/AMD_HIP.md); multi-GPU: [docs/MULTI_GPU.md](docs/MULTI_GPU.md).
- Setup's own tests run without a GPU or downloads: `python tools/test_setup_<name>.py` (for example
  `tools/test_setup_amd.py`, `tools/test_setup_choices.py`).
- Keep the docs' style: plain words, measured numbers with what they were measured on, no claims without a
  measurement.

## Contributing a change or report

- Search the open issues and pull requests first, and add to a thread that already covers your point.
- Open an issue with the form that fits (bug report, feature request or question).
- One change per pull request. Say what it changes and what it leaves alone.
- A new feature is opt-in, and the default path stays byte-identical to the last release. Say how you checked.
- Build every backend a file touches (CUDA, HIP, SYCL) before asking for review.
- Change a default only where you measured it faster, and show the numbers with what they were measured on.
- A report from hardware the maintainers do not have is welcome. Follow
  [docs/COMMUNITY_BENCHMARKS.md](docs/COMMUNITY_BENCHMARKS.md), compare against a same-day run of the build you are
  testing, and say what you did not test.
- Open test requests and the hardware that is wanted are listed in [docs/TEST_REQUESTS.md](docs/TEST_REQUESTS.md).

## Agent turn latency

Most of a turn's time is the model rereading its context and then writing; how long a local tool runs rarely matters.

- **Fewer turns:** make independent reads, searches and commands parallel tool calls in one turn. When several steps feed one decision, run them as one script or command instead of a turn each.
- **Small tool output:** everything a tool returns is reread on every later turn. Narrow it first with `rg`, `head`, `git diff --stat`, test summaries and exact line ranges; send long logs to a file and read only the part you need.
- **Stable prompt prefix:** do not add, remove or reconfigure tools or MCP servers mid-session unless that is the task. A changed tool list invalidates the cached prefix that every later turn rereads.
- **Compact long sessions:** at a natural boundary in a long task, commit usable work, write a short handoff (done, remaining, next step, evidence paths), and continue in a fresh session instead of carrying the whole history.
- **Code that calls a model API:** stream responses and act on each tool call as soon as its block is complete; reuse one long-lived HTTP client per process instead of reconnecting per request; put stable content (system prompt, tool definitions, reference material) first and per-request content (timestamps, IDs, user input) last so prompt caching hits.
