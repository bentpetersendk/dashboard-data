# Dashboard Data

This repository stores generated public dashboard data for BioHPC-related websites and services.

Generated JSON is kept separate from website source code so automated updates do not create unnecessary website commits, merge conflicts, or branch divergence.

## Directory Structure

```text
dashboard-data/
├── biohpc/
│   ├── stats.json
│   ├── projects.json
│   ├── users.json
│   └── storage.json
│
├── mjolnir/
│   ├── stats.json
│   ├── queue.json
│   ├── utilization.json
│   └── node_status.json
│
└── webservices/
    └── stats.json
```

Each top-level directory is a service namespace. Each JSON file should contain one focused data product for that service.

## BioHPC Metrics

The BioHPC website currently reads:

```text
https://raw.githubusercontent.com/bentpetersendk/dashboard-data/main/biohpc/stats.json
```

The scheduled GitHub Actions workflow in `.github/workflows/update_biohpc_stats.yml` runs hourly, can be triggered manually, generates this file from Airtable using `bentpetersendk/biohpc.github.io/scripts/update_stats.py`, and commits it only when the generated JSON changes.

## Adding Mjolnir Metrics

Add Mjolnir data under `mjolnir/` without changing existing BioHPC paths:

- `mjolnir/stats.json` for high-level summary metrics.
- `mjolnir/queue.json` for queue depth, wait time, and scheduler state.
- `mjolnir/utilization.json` for CPU, memory, GPU, and allocation utilization.
- `mjolnir/node_status.json` for node availability and health.

Use one workflow per source system when that keeps secrets, schedules, and failure modes independent. Keep JSON schemas stable once a public dashboard consumes them.

## Publishing Additional Services

Additional services should publish into their own top-level directory:

```text
service-name/stats.json
service-name/<specific-metric>.json
```

Prefer small, purpose-specific JSON files over one large shared file. Use repository or workflow variables for service names, output paths, branch names, and commit identity so workflows can be reused.

## Configuration

The BioHPC Airtable workflow supports these variables:

- `DASHBOARD_DATA_BRANCH` - target branch, default `main`.
- `BIOHPC_STATS_OUTPUT_PATH` - output path, default `biohpc/stats.json`.
- `BIOHPC_STATS_SOURCE_REPOSITORY` - repository containing `scripts/update_stats.py`, default `bentpetersendk/biohpc.github.io`.

Required secrets:

- `AIRTABLE_TOKEN`
- `AIRTABLE_BASE_ID`

Optional secrets:

- `BIOHPC_WEBSITE_TOKEN` - token used to checkout `bentpetersendk/biohpc.github.io` if that repository is private. If unset, the workflow falls back to `GITHUB_TOKEN`.

Optional variables:

- `AIRTABLE_VIEW`
- `AIRTABLE_TOTAL_USERS_REGISTERED_FORMULA`
- `AIRTABLE_ACTIVE_USERS_FORMULA`
- `AIRTABLE_APPROVED_USERS_FORMULA`
- `AIRTABLE_PENDING_USER_REQUESTS_FORMULA`
- `AIRTABLE_APPROVED_PIS_FORMULA`
- `AIRTABLE_PENDING_PI_REQUESTS_FORMULA`
- `AIRTABLE_ACTIVE_PROJECTS_FORMULA`
- `AIRTABLE_ORDERED_PROJECTS_FORMULA`

The published BioHPC `users` metrics distinguish historical adoption from current access:

- `users.registered` / "Total Users Registered" counts users whose `Account Status` indicates they have been approved and onboarded. The default formula counts `active`, `inactive`, `disabled`, `suspended`, `deactivated`, and `closed`.
- `users.active` / "Active Users" counts users whose `Account Status` is `active`.
- `users.approved` is retained for existing consumers and defaults to the active-access definition.

The workflow installs Python dependencies from the BioHPC website repository `requirements.txt` before running `scripts/update_stats.py`. If that file is missing or empty, the workflow fails with a clear configuration error instead of continuing to a later import failure.

The workflow maps optional Airtable filters from repository variables into the script environment. Leave optional variables unset to use the BioHPC website script defaults.
