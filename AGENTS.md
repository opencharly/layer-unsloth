# AGENTS.md — layer-unsloth

Standalone candy repo for the `unsloth` layer — a Tier 1 post-install layer that
layers a patched vLLM nightly wheel into a parent layer's pixi environment. The
candy lives in `charly.yml` at the repo root: the `env:` / `volume:` / `alias:`
sections, the pip-install and patch `run:` steps, the `check:` assertions, and
the embedded `skill:` entity projected into the marketplace corpus as
`/charly-jupyter:unsloth`.

Canonical files:

- `charly.yml` — the `unsloth:` candy entity and the `unsloth-skill:` skill
  entity.
- `patch_vllm_size_nodes.py` — the vLLM `_decompose_size_nodes` patch the plan
  applies.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-jupyter:unsloth` — the owning skill. The two-tier architecture, the
  post-pixi installs, and the vLLM patch. Load before editing or troubleshooting
  the layer.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:` and `agent-check:`, per-distro `distro:`
  arms, package/repo sections, service declarations). Load before editing any
  entity field or plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The candy's `plan:` `check:` steps are the functional evidence. This candy
  cannot be built standalone — it requires a parent pixi environment, so keep
  that contract explicit in any new check.
- The wheel URL is pinned to a specific vLLM nightly build; a bump must move the
  URL and the `check:` assertion together.

## Modify this repo

- Edit the `unsloth:` candy entity AND the `unsloth-skill:` skill entity in
  `charly.yml` together. The skill is the projected usage source, so a behaviour
  change not mirrored in the skill leaves the corpus stale.
- A change to the vLLM patch belongs in `patch_vllm_size_nodes.py` and in the
  matching `agent-check:` in the same change.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
