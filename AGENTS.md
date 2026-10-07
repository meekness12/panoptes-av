# AGENTS.md: Panoptes

## What we are building
Panoptes is a local-first, explainable endpoint security engine for Windows 11. It finds malicious files and processes, explains every verdict, and quarantines safely. It runs alongside Microsoft Defender, never instead of it. Solo developer plus coding agents.

## What we are not building
A cloud service, a port to other operating systems, a Defender replacement, a kernel driver before milestone M2 is stable, or anything that creates, modifies or spreads malware.

## Read first
1. docs/BRIEF.md
2. docs/DESIGN.md (phases, architecture, data model)
3. docs/adr/
4. docs/GLOSSARY.md
Before writing code, say which sub-phase you are on and restate its Done-when criteria.

## Architecture in one paragraph
An engine library: analyzers produce Findings, and a verdict engine fuses them into a 0-100 score and a label (CLEAN, LOW, SUSPICIOUS, MALICIOUS) with reasons. A CLI and a Windows service (panoptesd) wrap it. Untrusted files are parsed only in a separate low-privilege worker process with hard limits. SQLite stores results. Later phases add behavior monitoring (ETW), a VM sandbox, ML scoring and an optional kernel minifilter.

## Decisions already made (change only through an ADR)
- Python first; Rust only for profiled hot paths; the driver is C with the WDK.
- Analyzers return Findings and never decide the verdict.
- Real-time scanning fails open on timeout; quarantine restore fails closed.
- SQLite in WAL mode with versioned migrations.
- Sandbox work builds on CAPEv2 and runs only in an isolated VM.
- Local-first: no telemetry unless the user opts in.

## Non-negotiable rules
1. Never download, execute or commit real malware. Fixtures are the EICAR string (generated at runtime), benign synthetic files, or hashes only.
2. Anything that runs untrusted code runs only in the lab VM, never on the host.
3. Never auto-delete or auto-quarantine files in the Windows or Program Files folders without explicit user confirmation.
4. Analyzers make no network calls. Only the updater and the intel importer use the network.
5. Every analyzer survives corrupt, truncated, empty and huge files, and has a timing test.
6. No new dependency without checking its license and recording it in the third-party notices.
7. Ask instead of guessing when an acceptance criterion is ambiguous.

## Conventions
Python 3.12, src/panoptes layout, type hints everywhere, ruff, mypy (strict for the engine), pytest with hypothesis, at least 80% coverage on the engine. Conventional commits. One branch per sub-phase, for example phase-2.2-pe-analyzer.

## Commands (PowerShell, from the repo root)
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -e .[dev]
ruff check .
mypy src
python -m pytest

## Workflow for every task
1. Restate the acceptance criteria.
2. Write failing tests first.
3. Implement the smallest change that passes.
4. Run ruff, mypy and pytest.
5. Record any deviation as an ADR in docs/adr.
6. Summarize what changed and stop. Do not start the next sub-phase.

## Definition of done
Every Done-when item for the sub-phase passes by automated test, CI is green, and the docs are updated.

## Ask the human before
Adding a dependency, touching modules outside the sub-phase, running any unknown binary, or doing anything destructive to user files.
