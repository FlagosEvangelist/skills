# model-migrate-fl

Claude Code skill for migrating models from upstream vLLM into [vllm-plugin-FL](https://github.com/flagos-ai/vllm-plugin-FL) (pinned at vLLM v0.13.0).

## What it does

Automates the full migration workflow:

1. Clone upstream vLLM, locate model files
2. Copy-then-patch model code for 0.13.0 compatibility
3. Create config bridges, register models
4. Run unit tests, functional tests, benchmark, serve + request verification

Trigger with `/model-migrate-fl <model_name>` (e.g. `/model-migrate-fl qwen3_5`).

## Usage in vllm-plugin-FL

Claude Code requires skills to be placed under `.claude/skills/` in the project root. To use this skill in vllm-plugin-FL:

```bash
# From vllm-plugin-FL project root
mkdir -p .claude/skills
cp -r <path-to-this-repo>/skills/model-migrate-fl .claude/skills/
```

The resulting structure should be:

```
vllm-plugin-FL/
  .claude/
    skills/
      model-migrate-fl/
        SKILL.md              # Skill entry point (Claude Code reads this)
        references/
          procedure.md        # Step-by-step migration procedure
          compatibility-patches.md  # vLLM 0.13.0 patch catalog (P1-P12)
          operational-rules.md      # Communication, TaskList, bash rules
        scripts/
          validate_migration.py     # Automated code review
          benchmark.sh              # Benchmark verification
          serve.sh                  # Start model server
          request.sh                # Test request
```

## Key notes

- **`SKILL.md` is the entry point** -- Claude Code discovers and loads skills by scanning `.claude/skills/*/SKILL.md`
- **`allowed-tools` in SKILL.md** controls what tools the skill can use during execution
- **Scripts use relative paths** -- they reference `{{skill_root}}` which resolves to the skill's directory at runtime
- **The skill is idempotent** -- safe to re-run on an already-migrated model (overwrites with latest upstream)

## Compatibility patches

The skill maintains a catalog of vLLM 0.13.0 compatibility patches (`references/compatibility-patches.md`). Key patches include:

| Patch | Description |
|-------|-------------|
| P1 | Relative imports -> absolute imports |
| P2 | Config imports redirect to plugin bridges |
| P3-P5 | Remove/replace missing 0.13.0 APIs |
| P6 | Parent `__init__` reimplementation for VL models |
| P7 | MoE nested config (hf_config -> hf_text_config) |
| P8 | MergedColumnParallelLinear for merged projections with different sub-dimensions |
| P9-P10 | Mamba/hybrid model API differences |
| P11 | Plugin custom op import path deduplication |
| P12 | Complete `__init__` override attribute checklist |
