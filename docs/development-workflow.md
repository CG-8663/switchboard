# BMAD and ECC project setup

Verified on 20 September 2026. BMAD manages project planning and tracking; ECC supplies implementation, review, and verification practices. The root [AGENTS.md](../AGENTS.md) connects both to the existing Switchboard documents.

## Installed state

| Tool | Scope and version | Verification |
| --- | --- | --- |
| BMAD Core + BMad Method (`bmm`) | Project-local, 6.12.0 | Installer succeeded; all 29 skills listed in its manifest exist under `.agents/skills`; configuration resolver succeeded |
| ECC | Existing Codex user installation, 2.2.1, developer profile | All 276 recorded files exist; 273 match recorded hashes; three customized agent files were preserved |
| Node.js | Machine, v26.0.0 | Meets the BMAD installer's Node requirement |
| uv | Machine, 0.11.7 | Detected by installer; BMAD resolver executed offline successfully |

BMAD's installation receipt is `_bmad/_config/manifest.yaml`. Shared settings are in `_bmad/config.toml`; durable team overrides belong in `_bmad/custom/config.toml`. The configured knowledge directory is `docs`, with outputs under `_bmad-output`.

ECC's existing receipt is `~/.codex/ecc-install-state.json`. Its core project skills (`tdd-workflow`, `verification-loop`, `architecture-decision-records`, `agent-self-evaluation`, `configure-ecc`, and `ecc-guide`) are available in this session and match the installed release's source. The customized files are the `docs-researcher`, `explorer`, and `reviewer` agent configurations. No global files were changed during this setup.

ECC is installed through its existing managed-file layout, not the newer native Codex plugin lifecycle. The current upstream recommendation is a native plugin for fresh Codex installations, but upstream also warns against stacking installation methods. This setup retains the working installation. Its recorded hook consent is `declined`; automatic ECC hooks were not enabled. See the [official ECC installation guidance](https://github.com/affaan-m/ECC#installation).

## Working sequence

1. Use `bmad-help` to orient to the current stage. The original blueprint and reviewed build brief are inputs; a formal PRD and sprint plan have not yet been produced.
2. Use `bmad-prd` or `bmad-spec` to formalize requirements, followed by `bmad-architecture`, `bmad-create-epics-and-stories`, and `bmad-sprint-planning` as appropriate for this project.
3. Deliver one bounded story at a time. Use ECC's relevant implementation/test/review skills against that story's acceptance criteria; record results in the same BMAD implementation artifacts.
4. Verify changes before declaring the story done. Keep offline CI separate from live fleet/provider checks, and retain the stability gate before public release.

Use the existing [build brief](build-brief.md) and [integration plan](integration-plan.md) as the starting context. The next planning task is to turn the dry-run milestone into a PRD/spec with testable acceptance criteria.

New BMAD skills were installed after this Codex session started. Reopen the project or start a fresh Codex session to refresh skill discovery; then invoke `$bmad-help`. The installer, files, and supporting scripts have been verified; fresh-session menu discovery has not been observed in this session.

## Reproduce or maintain

From the project root, the pinned project install is:

```sh
npx --yes bmad-method@6.12.0 install \
  --directory . --modules bmm --tools codex \
  --user-name James --communication-language English \
  --document-output-language English --output-folder _bmad-output \
  --set bmm.project_knowledge=docs --no-shims --yes
```

Contributors can substitute their own name. Personal answers are ignored by Git. Commit the project skill files, shared configuration, and reviewed planning artifacts when the repository is initialized. Future upgrades should select an explicit reviewed version; running this command again is not needed for normal use. See the [official BMAD project-install guide](https://docs.bmad-method.org/start/install-bmad/).

Check resolved configuration with:

```sh
uv run _bmad/scripts/resolve_config.py --project-root "$PWD" \
  --key core --key modules.bmm
```

ECC remains a developer-tool prerequisite on each contributor's machine, not a runtime dependency of the Pinokio-distributed app. New Codex users without an existing ECC installation can follow its native setup:

```sh
codex plugin marketplace add affaan-m/ECC
codex plugin add ecc@ecc --json
codex plugin list --json
```

Do not run that fresh-install sequence over the existing managed ECC installation on this machine. A future migration must reconcile owned files and preserve customized settings first. Native hook trust remains controlled by Codex.

No GitHub repository, CI pipeline, runtime app, or public release was created by this tooling setup. LWC's scope question remains separate: the earlier readiness resolved to a parent Wiki, so no Wiki initialization or memory write was performed here.
