# ACS conformance run — what a Claude Code session shows a Guardian, and what it never does

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24060923.svg)](https://doi.org/10.5281/zenodo.24060923)

Raw records, the runner, and the report for one measurement of the OWASP **Agent Control Standard**
(`GenAI-Security-Project/agent-control-standard`):

> For four failure modes that occur **before** a tool call becomes a well-formed request — or inside
> the client's own retry and transport layers — does the ACS hook observe the call, and can a
> Guardian decide on it from what it received?

This repository holds the record, not a conclusion about whether ACS is any good. The policy question
(should a Guardian catch these?) is separate and is left to the people who own the specification.

Archived at **DOI [10.5281/zenodo.24060923](https://doi.org/10.5281/zenodo.24060923)** (all versions: [10.5281/zenodo.24060922](https://doi.org/10.5281/zenodo.24060922)); the archived snapshot is this repository at `08761f1`.

## Order of events

| When | What | Where |
|---|---|---|
| 2026-11-07, before any run | Four predictions and their falsification conditions, published so the order is checkable | [`PREREGISTRATION.md`](PREREGISTRATION.md) |
| 2026-11-07 | One wiring probe: a real client pointed at a controllable endpoint, recorded because it bears on P1 | [`PROBE-20261107.md`](PROBE-20261107.md) |
| 2026-11-07 | The measured runs, five scenarios × five repetitions, on a clean Linux runner | [`evidence/`](evidence/) |
| 2026-11-09 | **First public release of the results.** The upstream report is still open as [`#522`](https://github.com/GenAI-Security-Project/agent-control-standard/pull/522); this record does not wait on that. A revision after review would be published as a separate version | [`REPORT.md`](REPORT.md), [`evidence/`](evidence/) |

Nothing in `PREREGISTRATION.md` was edited after the runs. If a prediction failed, the file shows it
was written first.

## What was run

| | |
|---|---|
| Client | Claude Code `2.1.283` (`--print --permission-mode acceptEdits`) |
| Adapter | `adapters/src/acs/integrations/claude_code/adapter.py` at ACS `db457a84393bd80954c9b70b2271a719bf7c61cd` (`88acdfce` for the envelope-recording run — same tree plus one logging line), wired through a project-level `.claude/settings.json` on all six hook events |
| Guardian | `acs-example-guardian` on an OS-assigned port; signing material in the environment, no key file. Policy untouched — the recording run adds one statement that appends the request body to a file when `ACS_ENVELOPE_LOG` is set |
| Endpoint | `runner/stub_server.py` — Anthropic `/v1/messages` with five modes. Only the model is faked; the client, the adapter and the Guardian are the released ones |
| Runner | GitHub Actions `ubuntu-latest`, Python 3.12.16, `uv sync --locked`; workflow in `runner/acs-pilot.yml` |
| Repetitions | 5 per scenario, 5 scenarios |

The runner records `git status --porcelain` (empty) and the commit hash in every artifact, so the code
under test is exactly the commit named above. The five-by-five set was produced twice: first on
`db457a8` unmodified, then on `88acdfce` with the logging line, and the counts matched.

## Reproducing it

```bash
git clone https://github.com/simin-yuan/agent-control-standard acs && git -C acs checkout db457a84393b
cd acs/adapters && uv sync --locked && cd ../..
npm install -g @anthropic-ai/claude-code@2.1.283
for mode in complete length drop reset loop; do
  acs/adapters/.venv/bin/python runner/pilot.py \
    --acs acs/adapters --stub runner/stub_server.py \
    --mode "$mode" --reps 5 --out "evidence/$mode.json"
done
```

Each run gets its own temp workspace, its own Guardian, and its own endpoint port. The client's model
calls never leave the machine.

## What is in `evidence/`

One JSON per scenario. Per repetition: the client's exit code and stdout/stderr, the endpoint's request
log (including the `tool_result` the client echoed back — the only record of a call that never reached
the Guardian), the Guardian's step log, and the files the client left behind.

## Files and integrity

`REPORT.md` is the report (the same text as the upstream pull request); `runner/` holds the runner, the stub endpoint and the workflow; `evidence/` holds one JSON per scenario. `checksums.txt` lists the sha256 of every file in this repository except itself — `sha256sum -c checksums.txt` reproduces that from a clone.

## What this does not claim

- Not that ACS is insecure, and not that any failure here is exploitable. Where nothing executed, the
  client refused or the transport failed first; that is not a protection working.
- Not a conformance claim for any product, and not a judgement on any Guardian's policy.
- Not a generalisation beyond one client version, one adapter, and these scenarios. One host only.
- Not a specification violation. §6.4 governs what happens after a Guardian decides; this covers what
  arrives before there is anything to decide on.

## Licence and provenance

Two licences, split by what the files are: `REPORT.md`, `FINDINGS.md`, `PREREGISTRATION.md`, `PROBE-20261107.md` and `evidence/` are **CC BY 4.0** ([`LICENSE`](LICENSE)) — reuse is fine, credit is required. Everything in `runner/` is **Apache-2.0** ([`LICENSE-APACHE-2.0`](LICENSE-APACHE-2.0)), the same licence as the upstream project, so anything taken from it can go back there without a licence question.

The runner and the records are published so the measurement can be repeated; the report text is
CC BY-SA 4.0 to match the project's own content licence. Nothing here is derived from private
disclosures. `PREREGISTRATION.md` and `PROBE-20261107.md` are reproduced unedited.
---

Part of the agent reliability toolkit → [agent-faultbench](https://github.com/simin-yuan/agent-faultbench)
