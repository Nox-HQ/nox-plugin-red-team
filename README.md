> ## ⚠️ Archived — superseded by `nox attack` in core
>
> **Do not install this plugin.** Exploit hypothesis construction and
> evidence-backed validation live in the core CLI as `nox attack`, which builds
> hypotheses from static findings plus the AI inventory, exercises a running
> target, and reports traces backed by the evidence spine. `nox attack plan` is
> offline; `run` / `replay` / `regress` are ACTIVE, require `--authorize`, and
> never run as part of `nox scan`.
>
> Archived because the plugin duplicated that job and the duplicate was empty.
> `analyze` returns `{}` for every input:
>
> ```
> $ nox plugin call nox/red-team analyze
> {}
> $ nox plugin call nox/red-team analyze finding_id=x
> {}
> $ nox plugin call nox/red-team analyze cve_id=CVE-2021-44228
> {}
> $ nox plugin call nox/red-team analyze target=http://localhost:9999
> {}
> ```
>
> It also could not contribute to a scan even in principle: neither of its tools
> is named `scan` and neither declares `requires_scan_context`, which are the
> only two things `nox scan` invokes. Since nox 1.34.0 a scan says so out loud
> rather than silently registering it.
>
> One part worked correctly and is worth recording: `validate` was properly
> refused by the sandbox for declaring `network_hosts: *`, an active risk class
> and `needs_confirmation`. That is the policy engine doing its job.
>
> **Remove it:**
>
> ```bash
> nox plugin remove nox/red-team
> ```
>
> Then drop `nox/red-team` from `plugins.required` in `.nox.yaml` and use
> `nox attack --help`.
>
> Detail in [#43](https://github.com/Nox-HQ/nox-plugin-red-team/issues/43).

---

# nox-plugin-red-team

AI-powered attack path analysis and exploit validation plugin for [Nox](https://github.com/nox-hq/nox).

## Overview

Analyzes security findings to detect multi-step attack chains and optionally validates exploitability against live targets. Combines deterministic chain detection with opt-in LLM-powered attack path reasoning.

## Tools

- **analyze** — Detect attack chains from security findings (passive, read-only).
  Runs under the **default policy**: it reasons over findings the core scan
  already produced, so it needs no network access, mutates nothing, and requires
  no confirmation.
- **validate** — Validate exploitability against a target URL (active, needs
  confirmation). Requires an explicit policy opt-in, because it probes a live
  target.

The two declare their requirements separately (`ToolSafety`), so `validate`
being active does not gate `analyze`. Before per-tool safety existed, the
plugin could only declare the union of both, and `analyze` was rejected under a
passive policy despite needing nothing — this README described it as passive and
was right about the tool; the manifest simply could not express it.

## Rules

| ID | Description | Severity |
|----|-------------|----------|
| REDTEAM-001 | Multi-step attack chain detected | Critical |
| REDTEAM-002 | Privilege escalation path identified | High |
| REDTEAM-003 | Data exfiltration path identified | High |
| REDTEAM-004 | Container escape chain | High |
| REDTEAM-005 | Command injection chain | Critical |
| REDTEAM-006 | Validated: security header missing | Medium |
| REDTEAM-007 | Validated: authentication weakness exploitable | Critical |
| REDTEAM-008 | Validated: injection vulnerability confirmed | Critical |
| REDTEAM-009 | Validated: TLS misconfiguration confirmed | High |
| REDTEAM-010 | Validated: rate limiting absent | Medium |

## Safety

This plugin declares `RiskActive` with `NeedsConfirmation` and `NetworkHosts("*")`. The `validate` tool performs active HTTP probing and requires explicit user consent.

## Build

```bash
make build
make test
make lint
```

## License

Apache-2.0
