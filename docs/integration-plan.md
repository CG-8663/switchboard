# GitHub, Pinokio, and Jev integration

Recorded 20 September 2026. The user selected GitHub for development, CI/CD, and pull requests; Pinokio for distribution, installation, launch, and use; and Pinokio's interfaces for discovering and assessing reusable tools. Public contribution follows a stable, tested release, as specified in the [build brief](build-brief.md#public-release-milestone).

This records the selected direction and proposed contracts. No remote repository, workflow, launcher, or Switchboard runtime has been created yet.

## Responsibility split

| Component | Responsibility |
| --- | --- |
| GitHub | Source history, PR review, automated checks, versioned releases and contribution workflow |
| Pinokio launcher | Reproducible dependency installation, local start/update/reset, runtime status and Web UI entry point |
| Switchboard service and UI | Goal state, routing, resource reservations, tool catalog, attempts, evaluation, learning and observability |
| Pinokio tool adapter | Discover launcher-managed apps, resolve identity and readiness, invoke permitted lifecycle actions, and discover app-specific APIs |
| Jev adapter | Typed classification and ranking over eligible choices; separately defined evaluation operations |
| Herdr adapter | Agent harness sessions, state and results, within Switchboard's execution/resource contract |

Pinokio can host Switchboard and also expose other apps that Switchboard may use. Model serving, media APIs, and harness execution remain separate adapters. Discovering an app does not establish that its API can complete a particular task.

## Development and delivery

Proposed flow: branch → PR → offline CI and review → merge → versioned release candidate → Pinokio install/upgrade smoke tests → release available through Pinokio. Pinokio installations consume a tested version with an identifiable commit; ordinary PRs do not update the live fleet.

Use fixture-based CI without fleet credentials for every PR. It should cover eligibility, reservations, bounded retries, Jev response validation, budget limits, restart recovery, and adapter contracts. Live hardware and authenticated provider checks belong in a separate trusted workflow. External PR code must not inherit credentials or access to the private fleet.

Package the first release as an app launcher with app code separated from launcher scripts. Expected launcher surfaces are `install.js`, `start.js`, `update.js`, `reset.js`, `pinokio.js`, and `pinokio.json`, matched to the installed Pinokio examples when implemented. Start should expose the actual ready URL, use a local binding, and report readiness distinctly from process startup. Update and rollback must preserve user data; dependency reset must not implicitly erase goals, artifacts, or learned policy history.

A controller-only install should reuse existing model/tool services rather than download model weights as a side effect. Optional model runtimes declare their own dependencies and supported hardware. Keep local paths, secrets, database files, and model caches outside the distributable source set.

The official manual documents installation from a git repository URL and discovery through the GitHub `pinokio` topic. Treat that as the distribution mechanism to verify for the release, rather than deploying Switchboard onto GitHub's servers. See [Pinokio manual](https://desktop.pinokio.co/docs/#/?id=publish-your-script).

## Pinokio interfaces verified

The [official API manual](https://desktop.pinokio.co/docs/#/?id=api) was retrieved through its linked [Markdown source](https://desktop.pinokio.co/docs/README.md). It documents `script.start`, `script.stop`, and `script.return` for script orchestration, including passing arguments and returning results. It also documents `app.search` and `app.info` for native application metadata. Native application search is distinct from discovery of Pinokio launchers.

The installed Pterm documentation and an actual installed-app search establish the external discovery path for this machine. The following are the adapter boundaries to implement:

| Interface | Intended use | Evidence level |
| --- | --- | --- |
| `pterm search` | Search installed launcher descriptions and return identities and reported readiness | Executed successfully during this review |
| `pterm status <ref> --probe` | Confirm a selected app's readiness before using its API | Documented locally; not probed for every discovered app in this review |
| `pterm run <ref>` | Start through the existing launcher | Documented locally; no apps started in this review |
| `pterm stop <script> --ref <ref>` | Stop a specific owned script when a job's lifecycle requires it | Documented locally; no apps stopped in this review |
| `pterm upload` and discovered app API | Transfer a file when a remote app needs a source-local path, then invoke its operation | Contract to verify per app |
| `script.start/stop/return` | Compose workflows inside the Pinokio runtime | Verified in the official manual; integration smoke test pending |

Use returned `ref` values for identity and returned readiness URLs for API discovery. A remote app's filesystem path or loopback URL belongs to that machine, not necessarily to the caller. Resolve a reachable API address and remote input paths rather than synthesizing them from a hostname and port.

This closes the blueprint's uncertainty about whether an external discovery/control client exists: Pterm is installed and can query the running control plane. It does not prove an arbitrary public JSON-RPC endpoint or successful launch/stop behavior from every environment. A sandbox initially blocked the local control-plane connection with `EPERM`; the same read-only inventory succeeded with approved network access.

## Tool review and catalog

For each candidate, retain app identity and source revision, declared capabilities, API schema/version, input/output modalities, model requirements, hardware/context limits, locality, installation state, readiness freshness, cost/licensing constraints, and measured task results. Link these to the model, skill, and attempt records in the build brief.

Use explicit stages: discovered → metadata reviewed → API verified → smoke-tested → eligible. Mark incompatible, stale, or unavailable candidates with a reason. Jev can classify the reviewed metadata and rank eligible operations; executable commands and endpoint allowlists remain defined by adapters. Descriptions or README text cannot authorize arbitrary commands.

Initial installed-tool search results below are a dated discovery snapshot. Status is what Pterm reported on 20 September 2026; it is not an end-to-end task qualification or a live status dashboard.

| Candidate | Declared capability | Reported state | Integration consideration |
| --- | --- | --- | --- |
| MiniMax H3 | Video generation, UI/controller with delegated workers | Online, ready | Candidate video adapter; verify job submission, cancellation and output validation |
| ACE-Step 1.5 | Music generation | Starting, not ready | Treat launcher instance and separately deployed music services as distinct targets |
| ACE-Step UI | Music workspace and bundled backend | Offline | Identify whether the UI/backend duplicates an existing service before allocating resources |
| YUE2 GROOVE | Music/vocals experiments | Starting, not ready | Metadata says local generation is unverified and model weights are non-commercial; verify applicability before eligibility |
| Director Studio | Shot planning and asset workspace | Offline | Orchestration layer with its own backend target; discover actual operations rather than treating it as a model |
| J65 YuE2 Music | Planned music generation workers | Offline | Metadata says generation paused and compatibility unverified; keep ineligible until qualified |

The two bounded searches (`model inference` and `text generation`) returned the same six candidates. This is a useful initial sample, not a complete installed-app inventory, and it did not identify a qualified general text-model server. Search relevance is not proof of model capability. No registry downloads, app launches, or generation jobs were performed.

## Jev availability in this session

The existing Jev broker is running. The missing native tool here is a connection/configuration issue, with an additional sandbox restriction on the current client's Docker transport.

Evidence from this investigation:

1. The shared `typesafe-jev codex` command reported `ACTIVE`, aligned with `jev-1.13.0`, and `dedicated-client` access. Its fleet snapshot was approximately six seconds old at the first check.
2. This session's tool catalog contained no Jev/TypeSafe MCP tool. `codex mcp list --json`, inspected with credential fields suppressed, contained no Jev server. MCP servers must be registered to appear as native tools; see [official Codex MCP documentation](https://learn.chatgpt.com/docs/extend/mcp?surface=cli).
3. The existing fleet client uses `docker exec` to reach the private broker when its internal HTTP transport is not configured. That environment override was absent here. The default sandbox denied the Docker socket.
4. The approved read-only diagnostic confirmed the broker container was running and `/health` returned HTTP 200 with `ok: true`. No paid inference was sent.
5. Reading the telemetry code showed that its model value comes from the last successful typed-decision receipt, dated 19 September 2026. The fresh status measures current container state; it does not make that model receipt fresh. The broker requests `jev-latest`, while readiness compares against `jev-1.13.0`.

The `dedicated-client` label is a configured harness mapping, not a live test that the current shell can use that transport. A passing health endpoint proves broker reachability and process health, not current upstream authentication, balance, rate limits, or inference success. Those remain untested by this diagnostic.

The current broker also has a narrower API than Switchboard needs. It accepts bounded metadata and eligible candidates, discards caller-defined questions, and sends one fixed Choice question. It cannot currently classify arbitrary task text or evaluate generated output using Score/Noul. Preserve this narrow route-selection path and add separately reviewed classification/evaluation contracts with explicit payload and data-sharing limits. Do not send the full project or conversation through the route-selection broker.

For Switchboard, the application should call a configured Jev service adapter directly; adding a Codex MCP wrapper is a separate developer convenience. Do not give the shipped app a host Docker socket merely to copy the current diagnostic path. Its adapter needs connection tests, timeouts, response validation, permitted payloads, model-version reporting, and deterministic fallback. No MCP configuration, secret, or existing fleet service was changed during this investigation.
