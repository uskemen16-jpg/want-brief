# Want Brief

Want Brief keeps Jira, Todoist, and local work in one place. It combines task priorities, events, a weekly schedule, parallel time tracking, and daily check-ins.

## What you can do

- Review tasks in a board or an Eisenhower matrix, and keep project, person, and company records alongside them.
- Plan work around events and available time. The schedule shows capacity, at-risk items, and what-if comparisons.
- Run more than one timer at once. Pulse shows check-ins and time allocation; Today shows the current plan and earnings.
- Connect Jira and Todoist sources. Synchronization imports work and is read-only for those services.

## Local setup

You need Rust 1.85 or newer, Bun 1.3.14, and Docker Compose. Install the web dependencies once:

```sh
cd web && bun install --frozen-lockfile && cd ..
```

Start PostgreSQL and the API from the repository root:

```sh
make up
make api
```

In another terminal, start the web app:

```sh
make web
```

Open <http://127.0.0.1:5173>. Sign in with `BOOTSTRAP_PASSWORD`; its local default is defined in the Makefile.

The Makefile provides development defaults. To use values from a local `.env` file, copy `.env.example`, set your own values, and export them in each terminal before running `make`:

```sh
cp .env.example .env
set -a
. ./.env
set +a
```

The example password, token key, database credentials, and HTTP settings are for local use. Do not expose this setup to an untrusted network.

For a separate scratch database and web port, run `make api-dev` and `make web-dev` in two additional terminals. The scratch web app opens at <http://127.0.0.1:5174>.

## Existing installations

Rehearse the switch on a PostgreSQL copy. Back up the current database, restore it under a new name, set `DATABASE_URL` to the copy, and run `cargo run -- -migrate-only`. Start the Rust API against that copy with the existing `TOKEN_KEY` and `BOOTSTRAP_PASSWORD`. Check sign-in, saved sources, task sync, and the schedule before changing the live process.

For cutover, stop the Go process, take a fresh database backup, and start the Rust process with the same environment values. The Rust process reads the existing migration journal, session hashes, and sealed source tokens. Keep the backup and the previous `v1.0.3` release until the new process is accepted. To roll back, stop Rust and start `v1.0.3` against a restored backup in a separate database. Changes written after that backup need separate recovery.

## Sources and time

Add Jira or Todoist connections under Sources, then start a sync there. Jira uses a site URL and either a bearer token or an email and API token; Todoist uses a REST token. The default Jira query selects assigned, unfinished issues.

Each timer creates a time interval for an item. **Allocated time** sums intervals, including overlaps. **Wall-clock time** counts overlapping intervals only once.

## Checks and documentation

Run `make test` for Rust tests and TypeScript checks. Run `make lint`, `cd web && bun test`, `bun run lint`, and `bun run build` for the remaining local checks. To run database integration tests, set `TEST_DATABASE_URL` to an isolated database whose name starts with `wantbrief_issue92_`, then run `cargo test --tests -- --ignored --test-threads=1`.

See [architecture](docs/architecture.md) for the application layout, [glossary]() for domain terms, and [frontend notes](web/README.md) for web commands. Repository workflow conventions live in `docs/agents/`.

The [Want Brief design specification](docs/design/redesign.md) includes a responsive preview. The web interface uses Want Desyst v0.3.0 components and tokens.
