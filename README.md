# layer-unsloth

Patched vLLM runtime for [Unsloth](https://github.com/unslothai/unsloth) LLM
fine-tuning, layered into a parent pixi environment.

The `unsloth` candy is a **Tier 1 post-install layer**: it has no `pixi.toml`
and no dependencies of its own. It installs a vLLM cu130 nightly wheel into the
parent layer's pixi `default` environment (`pip install --no-deps`; the matching
runtime deps live in the parent's `pixi.toml`), then patches vLLM's
`_decompose_size_nodes` bug (upstream vllm-project/vllm#38360) so the
`torch.compile` graph passes stop crashing. It also exports
`UNSLOTH_SKIP_LLAMA_CPP_INSTALL=1` and `HF_HOME` (the HuggingFace cache, backed
by the `models` volume).

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `unsloth` (Tier 1 post-install; no `pixi.toml`) |
| Parent | a Tier 2 environment-owner candy (`jupyter-ml`, `unsloth-studio`) |
| Wheel | vLLM `0.19.1rc1.dev39` cu130 nightly (`pip install --no-deps`) |
| Patch | `patch_vllm_size_nodes.py` — the `_decompose_size_nodes` fix |
| Env | `UNSLOTH_SKIP_LLAMA_CPP_INSTALL=1`, `HF_HOME=~/.cache/huggingface` |
| Volume | `models` → `~/.cache/huggingface` |

## How to use it

This candy cannot be used standalone — compose it into an environment-owner
candy via the `candy:` field:

```yaml
my-ml-box:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-unsloth:v2026.239.1643'
```

Then, inside the built image:

```bash
~/.pixi/envs/default/bin/python -m pip show vllm   # 0.19.1
unsloth                                             # the fine-tuning CLI
```

The candy's `plan:` asserts the vLLM wheel in the pixi default env, the two
exported env vars, the pixi Python at the expected path, and an `agent-check`
that vLLM's compilation backend carries the `_decompose_size_nodes` fix rather
than the unpatched upstream code.

## Layout

- `charly.yml` — the `unsloth:` candy entity (the `env:` / `volume:` / `alias:`
  sections, the pip-install and patch `run:` steps, the `check:` assertions) and
  the embedded `unsloth-skill:` skill entity.
- `patch_vllm_size_nodes.py` — the vLLM `_decompose_size_nodes` patch applied by
  the plan.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-jupyter:unsloth`
- `/charly-jupyter:jupyter-ml` — Tier 2 parent (`candy: [llama-cpp, unsloth, jupyter-mcp]`)
- `/charly-jupyter:unsloth-studio` — Tier 2 parent (`candy: [llama-cpp, unsloth]`)
- `/charly-jupyter:llama-cpp` — sibling Tier 1 candy (llama.cpp binaries)
- `/charly-languages:python-ml` — core ML environment (Tier 2)
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
