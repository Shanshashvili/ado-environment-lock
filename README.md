# Pipeline Lock

Simple Azure DevOps pipeline setup to serialize runs per environment (`dev`, `qa`, `prod`) using run tags.

## Purpose

This project provides a lightweight "environment lock" mechanism so that only one active run for the same target environment can proceed at a time.

## How it works

The pipeline:

1. Runs manually (no CI trigger).
2. Receives `targetEnv` as input.
3. Registers a run tag: `env-<targetEnv>`.
4. Polls Azure DevOps Build API for active runs with the same tag.
5. Waits until the current run is the oldest active run for that environment.
6. Continues execution (currently includes a demo sleep step).

## Project structure

- `environment-lock-manager.yml` — main pipeline definition.
- `templates/variables-oboba.yml` — shared variables/parameters template.
- `README.md` — project overview, usage, and pipeline lock behavior documentation.
- `LICENSE` — MIT license for repository usage and distribution.

## Parameters

- `targetEnv` (string): `dev` | `qa` | `prod`.

## Variables currently used

From the variables template, the current pipeline uses:

- `agentPool`
- `targetEnv`

## Prerequisites

- Azure DevOps pipeline with **Allow scripts to access the OAuth token** enabled.
- Agent image with `bash`, `curl`, and `python` (or `python3`) available.

## How to use

1. Create a new Azure DevOps pipeline and point it to `environment-lock-manager.yml`.
2. In pipeline settings, enable **Allow scripts to access the OAuth token**.
3. Make sure the selected agent pool has `bash`, `curl`, and `python`/`python3`.
4. Run the pipeline manually.
5. Choose `targetEnv` (`dev`, `qa`, or `prod`) and start the run.
6. Verify logs:
	- lock tag is registered (`env-<targetEnv>`),
	- run waits if an older run for the same environment is active,
	- run continues when it becomes first in queue order.

## Notes

- The lock is based on build tags and active run status (`notStarted`, `inProgress`).
- Timeout behavior is controlled in the wait step (`max_attempts` and `sleep_seconds`).
