# Linkly — Master Plan

Nov 25, 2026

This is the single source of truth for the Linkly project: eleven gates, each with entry conditions, tasks and exit conditions. Never start a gate until the previous gate's exit conditions are all ticked.

## Resuming after a break

Follow these steps at the start of every session, whether the break was a day or a month.

1. Open the **Status tracker** below and find the gate marked *In progress* (or *Blocked*).
2. Read the latest row in the **Session log** at the bottom. Its "Next step" column is where you start.
3. Go to that gate's section and re-check its entry conditions. If you changed anything in an earlier gate since your last session, confirm the entry conditions still hold.
4. Work through the unticked tasks in order.
5. Before you stop, add a Session log row with what you did, the exact next step and any blockers. Tick every task and exit condition you finished, and update the tracker.

**Ground rules**

- A gate is *Done* only when every exit condition is ticked. Unfinished tasks that don't affect exit conditions can move to the backlog (GitHub Issues).
- If a later gate shows that a requirement or design decision was wrong, don't patch around it. Update `docs/02-requirements.md` or the relevant ADR, add an entry to its change log, then trace the change forward.
- Commit small and often. Every session should end with work pushed, even if it's on a branch.

**If you fall behind**

- G0–G8 is the core project. G9's optional tasks and all of G10 are bonuses.
- Cut in this order: G10, Datadog, the EKS test, Tempo (keep metrics and logs), the Lambda probe.
- Time-box tasks: if one runs past twice its share of the gate estimate, log it in the Session log and decide whether to simplify, defer or push on.

## Status tracker

Update the Status column whenever a gate starts or finishes. The whole project is about 255–380 hours: roughly 14–21 weeks at 3 hours a day, 6 days a week (18 hours a week). The core project, G0–G8, is about 220–330 hours, or 12–18 weeks. Re-estimate after G0: if it took more than 40 hours, add about 30% to the remaining gates.

| Gate  | Name | Status | Estimate (hours) | Weeks at 18 h/week | Actual hours | Started | Finished | Main output |
| ---   | --- | --- | --- | --- | --- | --- | --- | --- |
| G0    | Project and lab setup | In Progress | 25–40 | 1.5–2 |  |  |  | Repo, Jenkins VMs via Terraform + Ansible, ADR-0001 |
| G1    | Requirements | Not started | 8–12 | 0.5–0.7 |  |  |  | `docs/02-requirements.md` |
| G2    | System design | Not started | 20–30 | 1–1.7 |  |  |  | `docs/03-design.md`, ADRs, diagrams, `api/openapi.yaml` |
| G3    | Core link service | Not started | 40–60 | 2.2–3.3 |  |  |  | Working API, tests, Jenkins CI pipeline |
| G4    | Cache, analytics and abuse | Not started | 30–45 | 1.7–2.5 |  |  |  | All v1 features, tag `v0.1.0` |
| G5    | Platform and deploy | Not started | 40–60 | 2.2–3.3 |  |  |  | k3s cluster, Argo CD, public redirects, S3 backups |
| G6    | Observability and SLOs | Not started | 20–30 | 1.1–1.7 |  |  |  | `docs/04-slos.md`, dashboards, alerts, external probe |
| G7    | Load testing and capacity | Not started | 10–15 | 0.6–0.8 |  |  |  | `docs/05-capacity.md` |
| G8    | Chaos and postmortems | Not started | 25–35 | 1.4–1.9 |  |  |  | `docs/postmortems/` |
| G9    | Hardening and v1.0 | Not started | 15–25 | 0.8–1.4 |  |  |  | Tag `v1.0.0`, retrospective |
| G10   | Kafka migration | Not started | 20–30 | 1.1–1.7 |  |  |  | Click events on Kafka, tag `v1.1.0` |

## Repository layout

This is a reference map, not a setup step: create each folder when you first put something in it, and check here so the structure stays consistent. Documents are numbered in the order you create them. Export this plan as Markdown to `docs/00-master-plan.md` and the product brief to `docs/01-product-brief.md`.

```
linkly/
├── README.md                  # what it is, how to run it in 5 minutes
├── pyproject.toml             # dependencies + ruff, mypy, pytest config
├── uv.lock                    # pinned dependency versions
├── Jenkinsfile                # declarative CI pipeline, G0–G3
├── Makefile                   # make test, make run, make lab-up ...
├── docs/
│   ├── 00-master-plan.md      # this doc
│   ├── 01-product-brief.md    # Linkly — Product Brief
│   ├── 02-requirements.md     # G1
│   ├── 03-design.md           # G2
│   ├── 04-slos.md             # G6
│   ├── 05-capacity.md         # G7
│   ├── 06-retrospective.md    # G9
│   ├── adr/                   # NNNN-short-title.md, from G0 on
│   ├── diagrams/              # Mermaid or Excalidraw sources, G2
│   ├── runbooks/              # one per alert and per procedure, G0–G6
│   └── postmortems/           # one per experiment or incident, G8
├── api/openapi.yaml           # API contract (source of truth), G2
├── src/linkly/
│   ├── main.py                # FastAPI app entry point
│   ├── worker.py              # click-event worker entry point
│   ├── api/                   # routers: HTTP parsing, validation, status codes
│   ├── services/              # business logic, no HTTP or SQL
│   ├── repositories/          # Postgres and Redis access behind Protocols
│   ├── domain/                # core types and domain errors
│   └── config.py              # settings from environment variables
├── tests/
│   ├── unit/
│   ├── integration/           # real Postgres/Redis via testcontainers
│   ├── contract/              # Schemathesis tests against openapi.yaml
│   └── e2e/
├── migrations/                # Alembic versions
├── deploy/
│   ├── terraform/
│   │   ├── proxmox/           # lab VMs: Jenkins (G0), k3s nodes (G5)
│   │   └── aws/               # S3 buckets, budget alarm, Lambda probe
│   ├── ansible/               # OS setup, Jenkins, k3s install
│   ├── jenkins/               # JCasC config (Jenkins as code)
│   ├── helm/                  # charts for api and worker, G5
│   └── argocd/                # Argo CD applications (GitOps), G5
├── observability/             # dashboards (JSON), alert rules, G6
├── loadtest/                  # k6 scripts, G7
└── chaos/                     # Chaos Mesh experiments, G8
```

## Tech stack

Decided on 2026-11-25. Each choice gets an ADR in `docs/adr/` during G0 or G2. Budget: $20 a month of cloud spend, plus a 2-node Dell PowerEdge R715 lab running Proxmox and Tailscale.

| Layer | Choice | Why |
| --- | --- | --- |
| Language and framework | Python 3.12, FastAPI, Pydantic, uv | Async I/O for a redirect-heavy service; validation built in |
| Data access | SQLAlchemy 2.0 (async), Alembic | Industry standard; use Core queries on the redirect hot path |
| API contract | Spec-first `api/openapi.yaml`, Spectral lint, Schemathesis contract tests | The spec is the source of truth; CI fails if code drifts from it |
| Database | PostgreSQL via the CloudNativePG operator | Replicas, failover and S3 backups managed in Kubernetes |
| Cache and rate limiting | Redis | One tool for cache, rate limits and (at first) the queue |
| Event queue | Redis Streams, then Kafka (Strimzi) in G10 | Start simple; migrating to Kafka is its own gate |
| Lab provisioning | Terraform (Proxmox provider), Ansible | Matches the internship team's stack |
| Compute | k3s (CNCF-certified) on Proxmox VMs across both R715 hosts, with its bundled Flannel, Traefik and ServiceLB replaced by Cilium, MetalLB and a standard ingress controller | Same Kubernetes API as EKS, a real multi-node cluster at no cloud cost, and production-style add-ons for bare metal |
| Packaging | Docker, Helm | Standard |
| CI | Jenkins (controller and agent in their own VMs), declarative `Jenkinsfile`, JCasC | Matches the team; Jenkins itself is rebuilt from code |
| Image registry | GitHub Container Registry | Free for public images |
| CD | Argo CD (GitOps) | The cluster pulls from Git, so nothing needs inbound access to the lab |
| Ingress | Cloudflare Tunnel for public redirects only; Tailscale for everything else | No ports opened on the home network |
| Observability | Prometheus, Grafana, Alertmanager, Loki, Tempo; Datadog as a stretch in G9 | Matches the team; self-hosting teaches more |
| AWS (within $20) | S3 (Terraform state, Postgres backups), Lambda external probe, budget alarm; optional short-lived EKS in G9 | Real AWS experience for a few dollars |
| Load and chaos | k6, Chaos Mesh | High load generation; Kubernetes-native experiments |

**Known risks accepted up front** (record each in an ADR):

- **etcd quorum on two physical hosts.** Three k3s server nodes run on two hosts, so losing the host with two of them loses quorum. Accepted for v1 and tested in G8. Fix later with a third small machine.
- **Jenkins and public pull requests.** Jenkins builds pull requests only from users with write access, so forked PRs can't run code in the lab. Keep the repository private until G9 if in doubt.
- **GitHub webhooks can't reach Jenkins** behind Tailscale. Start with SCM polling; optionally expose only the webhook path through Cloudflare Tunnel.
- **Electricity.** The R715s draw a lot of power. Shut one host down when you're not working on the project.

**Remote lab access**

The servers are at a remote location and reachable only through Tailscale, so losing remote access means losing the lab until someone is on site.

- **Tailscale runs on both Proxmox hosts themselves**, not only inside VMs, so a broken VM or cluster can never lock you out. One host (or a small VM) acts as a Tailscale subnet router for the lab network. Use Tailscale ACLs and MagicDNS.
- **Host-level changes are high risk.** Changes to Proxmox networking, host firewalls or Tailscale itself need a rollback plan (for example, a scheduled job that reverts the change unless you cancel it) and are best done when someone on site could help.
- **Hardware failures** (a disk, a power supply) need a person on site. Record who to contact in `docs/runbooks/lab-access.md`.
- **Load and chaos tools run on VMs inside the lab**, not from your laptop over Tailscale, so neither your connection nor the remote site's uplink skews results.

## Engineering standards

These apply from G0 onwards, to every change.

- **Commits:** Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`), small and focused.
- **Pull requests:** a PR template with what changed, why, how it was tested, and which FR, NFR or ADR it relates to. Review your own diff before merging.
- **Definition of done for a PR:** CI green, tests added or updated, docs and runbooks updated, no new TODOs without a linked issue.
- **Versioning:** semantic versioning tags and a `CHANGELOG.md` updated with every release.
- **Docs as code:** design docs, ADRs, runbooks and postmortems live in the repo and change through PRs like code.

## G0 — Project and lab setup

**Goal:** a clean repository, a working local toolchain and a Jenkins CI server built entirely from code, so every later gate starts from a known state.

**Entry conditions**

- [ ] You have read `docs/01-product-brief.md` once, end to end.
- [ ] Both R715 hosts are reachable over Tailscale, and Proxmox is healthy on each.

**Tasks: repository and local tools**

- [ ] Create the `linkly` GitHub repository with `README.md`, `.gitignore`, a license and a `docs/` folder. Create every other folder only when you first add something to it, following the Repository layout section.
- [ ] Export the product brief and this plan as Markdown into `docs/`.
- [ ] Install and verify: Git, Docker, Python 3.12+ with uv, `pre-commit`, Terraform, Ansible, kubectl, Helm.
- [ ] Write ADR-0001 (Python + FastAPI) and ADR-0002 (lab on Proxmox, plus AWS within a $20 budget), using the template: Context, Decision, Alternatives, Consequences.
- [ ] Turn on branch protection for `main`: pull requests only, no direct pushes.
- [ ] Create a GitHub Project board with columns Backlog, This gate, In progress, Done.

* [ ] Inventory the lab: CPU, RAM and disk on each R715, and a VM sizing budget for Jenkins, the k3s nodes and headroom. Record it in `docs/runbooks/lab-inventory.md`.

**Tasks: remote access and recovery**

- [ ] Confirm Tailscale runs on both Proxmox hosts, and set up a subnet router, ACLs and MagicDNS for the lab network.
- [ ] Write `docs/runbooks/lab-access.md`: how to reach each host, how to recover if Tailscale on a host breaks, and who can help on site.

**Tasks: AWS foundation**

- [ ] Secure the AWS account: MFA on root, an IAM user or SSO role for daily work, no root access keys.
- [ ] Create an AWS Budget with alerts at $10 and $18 a month.
- [ ] Create an S3 bucket for Terraform state (versioning on, public access blocked) and use it as the backend for `deploy/terraform/`.

**Tasks: Jenkins in the lab**

- [ ] Write Terraform in `deploy/terraform/proxmox/` that creates two VMs: `jenkins-controller` and `jenkins-agent`.
- [ ] Write Ansible playbooks in `deploy/ansible/` for base OS setup (updates, users, SSH hardening, firewall), the Jenkins controller and the agent (Docker installed).
- [ ] Configure Jenkins with JCasC (`deploy/jenkins/`): plugins, credentials pulled from secrets, the GitHub Branch Source plugin, and the agent connection.
- [ ] Restrict pull request builds to users with write access, so forked PRs can't run code in your lab.
- [ ] Set up SCM polling (or a Cloudflare Tunnel exposing only the webhook path) and build status reporting back to GitHub.
- [ ] Commit a "hello world" `Jenkinsfile` that checks out the repo and runs `uv --version`.
- [ ] Write `docs/runbooks/rebuild-jenkins.md` and ADR-0003 (Jenkins for CI, Argo CD for CD).

**Exit conditions**

- [ ] The repository is pushed, and every folder created so far follows the Repository layout section.
- [ ] ADR-0001 to ADR-0003 are merged through pull requests.
- [ ] Destroying and recreating the Jenkins VMs from Terraform + Ansible + JCasC gives a working Jenkins with no manual clicks. Test this once.
- [ ] A pull request shows a green Jenkins status check on GitHub, and branch protection requires it.
- [ ] The AWS budget alert exists, and Terraform state lives in S3.
- [ ] The first Session log row for G0 is written.

## G1 — Requirements

**Goal:** a numbered, measurable requirements doc that every later decision, test and alert can trace back to.

**Entry conditions**

- [ ] G0 is Done.
- [ ] The product brief is in the repository and you have not changed it. Treat it as the customer's words.

**Tasks**

- [ ] Extract functional requirements as FR-1, FR-2, …, grouped by area: link creation, redirect, management, analytics, accounts and API, abuse.
- [ ] Extract non-functional requirements as NFR-1, NFR-2, … Each needs a number and a way to measure it (for example, "p99 redirect latency under 100 ms, measured at the load balancer over 28 days").
- [ ] Map every stakeholder quote in the brief to at least one requirement, or note why you rejected it.
- [ ] Resolve conflicting stakeholder wishes (exact analytics vs. never slowing redirects, "never down" vs. the $500 budget). Write each resolution as a numbered assumption, A-1, A-2, …
- [ ] Answer every open question in the brief as an assumption.
- [ ] Do the back-of-envelope estimates for launch and for 2 years: average and peak requests per second for redirects and writes, storage after 1 and 5 years, cache size to hold the hottest 20% of links, and bandwidth. Show your working.
- [ ] Write an explicit scope list: in v1, deferred, never.
- [ ] Add a change log section at the bottom of the doc.
- [ ] Get a review (Claude, a mentor or a classmate) and address every comment.

**Exit conditions**

- [ ] `docs/02-requirements.md` is merged.
- [ ] Every NFR has a number, a unit and a measurement method. No adjectives like "fast" or "reliable" without numbers.
- [ ] Every stakeholder quote and every open question is covered by a requirement or an assumption.
- [ ] Estimates are written out with their arithmetic, at launch and 2-year scale.
- [ ] Review feedback is resolved, and the commit is tagged `requirements-v1`.

## G2 — System design

**Goal:** a reviewed design that meets every requirement, with decisions and trade-offs written down before any production code exists.

**Entry conditions**

- [ ] G1 is Done and `requirements-v1` is tagged.
- [ ] From here on, requirement changes go through the change log in `docs/02-requirements.md`.

**Tasks**

- [ ] Start `docs/03-design.md` with these sections: Context, Goals and non-goals, Proposed design, Alternatives considered, Failure modes, Scaling plan, Risks, Open questions.
- [ ] Draw a C4 context diagram (Linkly as one box, plus customers, visitors, support and the malicious-URL service).
- [ ] Draw a C4 container diagram (API, worker, Postgres, Redis, queue, and how they connect).
- [ ] Draw sequence diagrams for: creating a link, a redirect with cache hit and with cache miss, editing a destination (including cache invalidation), and a click event flowing to analytics.
- [ ] Design the data model: an ERD with every table, key and index, and the reason for each index.
- [ ] Write `api/openapi.yaml` covering every FR, including error responses and status codes. This file is the source of truth: code must conform to it, never the other way round.
- [ ] Write ADRs for at least: short-code generation, cache strategy, 301 vs. 302 redirects, sync vs. async click tracking, visitor IP handling, deleted-code reuse, per-customer rate limiting, the event queue (Redis Streams now, Kafka later), and running k3s across two physical hosts (the etcd quorum risk).
- [ ] Fill in a failure-modes table: component, how it fails, user impact, how you detect it, mitigation.
- [ ] Write the scaling plan: what changes at the 2-year numbers (read replicas, partitioning, CDN, multi-region).
- [ ] Add a traceability table mapping each FR and NFR to the design element that satisfies it.
- [ ] Get a design review and address every comment.

**Exit conditions**

- [ ] `docs/03-design.md` is merged, with diagram sources in `docs/diagrams/`.
- [ ] `api/openapi.yaml` passes a linter such as Spectral.
- [ ] At least 9 design ADRs (on top of ADR-0001 to ADR-0003) have status Accepted.
- [ ] Every FR and NFR appears in the traceability table, and every component appears in the failure-modes table.
- [ ] Review feedback is resolved, and the commit is tagged `design-v1`.

## G3 — Core link service

**Goal:** a well-structured, well-tested API for creating, redirecting and managing links, built against the design.

**Entry conditions**

- [ ] G2 is Done and `design-v1` is tagged.
- [ ] The OpenAPI spec and data model are stable. Any change needs an ADR or a design-doc change log entry.

**Tasks**

- [ ] Scaffold the layered structure: `handler` → `service` → `repository`, with `domain` types. Services depend only on interfaces (typing.Protocol classes), so tests can swap in fakes. Use FastAPI dependency injection to wire real implementations.
- [ ] Extend the `Jenkinsfile` into the full CI pipeline, run on every pull request: ruff (lint and format check), mypy in strict mode, pytest unit and integration tests, Schemathesis contract tests against `api/openapi.yaml`, a check that the FastAPI-generated spec matches it (oasdiff), image build and push to GitHub Container Registry, and secret scanning (gitleaks).
- [ ] Add pre-commit hooks for formatting and linting.
- [ ] Write the schema as versioned Alembic migrations, using SQLAlchemy 2.0 (async) for data access and Pydantic models for request and response validation.
- [ ] Build the short-code generator test-first (TDD), following your ADR.
- [ ] Implement link creation with full validation: URL format and length, alias rules, reserved words, self-links.
- [ ] Implement redirects (reading straight from Postgres for now), plus the not-found page.
- [ ] Implement management: list, search, edit destination, set or clear expiry, delete (following your deletion ADR).
- [ ] Implement accounts and API keys. Store keys hashed, and make sure customers can only reach their own links.
- [ ] Enforce the free-tier limit of 50 links a month.
- [ ] Define typed domain errors mapped to HTTP responses in one place, with a consistent error body.
- [ ] Write unit tests for the service layer, integration tests against real Postgres (testcontainers-python) and a few end-to-end API tests.
- [ ] Add `docker compose up` for local development, and a README quick-start.

* [ ] Store customer passwords with a slow hash (argon2 or bcrypt), and use short-lived sessions for the dashboard.
* [ ] Set up Renovate or Dependabot so dependency updates arrive as pull requests that Jenkins tests.

**Exit conditions**

- [ ] Every FR for creation, redirect, management and accounts has at least one passing test, and the traceability table links them.
- [ ] CI is green on `main`, and branch protection requires it.
- [ ] Service-layer test coverage is at least 70%, measured with pytest-cov, and mypy --strict passes.
- [ ] A fresh clone runs locally in under 5 minutes by following the README. Test this from a clean directory.
- [ ] Gitleaks finds no secrets in the repository history.

## G4 — Cache, analytics and abuse handling

**Goal:** every v1 feature is complete, and the redirect path is fast and independent of analytics.

**Entry conditions**

- [ ] G3 is Done.

**Tasks**

- [ ] Add a Redis cache on the redirect path using your cache ADR, including invalidation when a destination is edited, expired or disabled.
- [ ] Make redirects fall back to Postgres when Redis is unavailable, with timeouts on every cache call.
- [ ] Publish click events to a queue (Redis Streams, per your queue ADR) without blocking the redirect.
- [ ] Build the worker that consumes click events and writes hourly and daily aggregates, with country, referrer and device.
- [ ] Anonymize or hash visitor IPs before storage, following your ADR.
- [ ] Filter bots and link-preview crawlers according to your assumption from G1.
- [ ] Implement expired and disabled pages.
- [ ] Build the stats endpoints for customers.
- [ ] Build support endpoints to disable links and suspend accounts, writing an audit log entry for every action.
- [ ] Check new destinations against the malicious-URL list, with the fallback behavior you chose in G1 when that check is unavailable.
- [ ] Add per-API-key rate limiting so one customer's bulk import can't slow down others.

* [ ] Implement account deletion that removes a customer's links and data within 30 days (Legal's requirement), and document what remains in backups and for how long.
* [ ] Add a scheduled job that rolls raw click data older than one year into daily summaries.

**Exit conditions**

- [ ] Every v1 FR maps to at least one passing test in the traceability table.
- [ ] A test proves redirects still succeed with the queue or worker stopped.
- [ ] A test proves an edited destination takes effect within the time your NFR states.
- [ ] A test proves every support action writes an audit log entry.
- [ ] Release `v0.1.0` is tagged, with a CHANGELOG entry.

## G5 — Platform and deploy

**Goal:** Linkly runs on a k3s cluster in your lab, deploys automatically through GitOps, serves public redirects safely and backs up its data to S3.

**Entry conditions**

- [ ] G4 is Done and `v0.1.0` is tagged.
- [ ] Jenkins CI from G0 and G3 is green and pushing images to GitHub Container Registry.

**Tasks: cluster**

- [ ] Extend `deploy/terraform/proxmox/` to create the k3s VMs: three server nodes (split 2 + 1 across the two R715 hosts) and at least one worker per host.
- [ ] Write Ansible playbooks that install k3s with embedded etcd, with the bundled Flannel, Traefik and ServiceLB turned off, then join the nodes.
- [ ] Write ADRs for k3s vs. kubeadm (including why its default add-ons were swapped out), and for the etcd quorum risk: which host holds two servers, and what happens if it fails.
- [ ] Install core add-ons with Helm: Cilium (networking and NetworkPolicies), MetalLB (load balancer IPs on your lab network), an ingress controller or Gateway API implementation, cert-manager, a storage class for persistent volumes (for example, Longhorn), Sealed Secrets or External Secrets.

**Tasks: data**

- [ ] Deploy Postgres with the CloudNativePG operator: one primary and one replica on different physical hosts.
- [ ] Configure continuous backups to an S3 bucket (Terraform in `deploy/terraform/aws/`, with a lifecycle rule to expire old backups).
- [ ] Deploy Redis with persistence turned on.

**Tasks: application delivery**

- [ ] Write multi-stage Dockerfiles that run as a non-root user, with small final images (a python:3.12-slim base, served by uvicorn, or gunicorn with uvicorn workers).
- [ ] Write Helm charts in `deploy/helm/` for the API and worker: readiness and liveness probes, resource requests and limits, graceful shutdown, Pod Disruption Budgets, anti-affinity across hosts.
- [ ] Install Argo CD and define the Linkly applications in `deploy/argocd/`.
- [ ] Add a Jenkins stage that, after a merge to `main`, updates the image tag in the Helm values and commits it. Argo CD then deploys it.
- [ ] Run database migrations as an Argo CD pre-sync hook (a Kubernetes Job) before each rollout.
- [ ] Expose only the redirect path publicly through Cloudflare Tunnel. Keep the management API, Argo CD, Grafana and Jenkins on Tailscale.
- [ ] Document and test rollback: a `git revert` of the image tag, and Argo CD's own rollback.
- [ ] Track monthly AWS spend against the $20 budget.

* [ ] Register a domain on Cloudflare and point the tunnel at it.
* [ ] Schedule k3s etcd snapshots to S3 and Proxmox backups of the Jenkins VMs, and test restoring one of each.

**Exit conditions**

- [ ] The whole platform rebuilds from an empty Proxmox with Terraform, Ansible and Argo CD, following `docs/runbooks/rebuild-cluster.md`.
- [ ] A merged pull request reaches the cluster with no manual steps.
- [ ] A rollback has been run and timed, and the time is recorded in `docs/runbooks/rollback.md`.
- [ ] A Postgres restore from S3 into a fresh cluster has been tested once.
- [ ] Public redirects work through Cloudflare, and the management API is not reachable from the internet (verified from outside your network).
- [ ] Every workload has probes, resource limits and replicas spread across both hosts.

## G6 — Observability and SLOs

**Goal:** you find out about problems before customers do, and alerts page only for what can't wait.

**Entry conditions**

- [ ] G5 is Done and Linkly is running in a deployed environment.

**Tasks**

- [ ] Expose Prometheus metrics with prometheus-client: request rate, errors and latency histograms per endpoint; cache hit ratio; queue lag; worker throughput; DB pool usage.
- [ ] Add structured JSON logs with a request ID on every line.
- [ ] Add OpenTelemetry tracing (its FastAPI, SQLAlchemy and Redis instrumentation packages) across API → cache → database → queue → worker.
- [ ] Build Grafana dashboards: a service overview (RED method) and a dashboard per component.
- [ ] Write `docs/04-slos.md`: an SLI, SLO target, window and error budget for each relevant NFR from G1.
- [ ] Create multi-window burn-rate alerts for each SLO.
- [ ] Sort every alert into "page now" or "ticket for the morning", following the on-call engineer's quote in the brief.
- [ ] Write a runbook in `docs/runbooks/` for every alert: what it means, how to confirm it, first steps.
- [ ] Add an external synthetic probe: an AWS Lambda on a schedule (Terraform in deploy/terraform/aws/) that requests a known public redirect through Cloudflare, records success and latency, and alerts when it fails.

* [ ] Route paging alerts to your phone (for example, ntfy or a free PagerDuty plan), then run a one-week on-call rotation for yourself, logging every page and whether it was actionable.

**Exit conditions**

- [ ] `docs/04-slos.md` is merged, and every latency or availability NFR has an SLO with a working SLI query.
- [ ] Every alert links to a runbook.
- [ ] Injected errors (for example, a flag that fails 5% of redirects) trigger the expected alert within the expected time.
- [ ] Dashboards and alert rules are saved in `observability/` and load from the repository.
- [ ] One request can be followed end to end in a trace.

## G7 — Load testing and capacity

**Goal:** real measurements of how much load Linkly handles, compared with your G1 estimates.

**Entry conditions**

- [ ] G6 is Done. You need dashboards to read load-test results.

**Tasks**

- [ ] Write k6 scripts in `loadtest/` for four scenarios: steady launch traffic, evening peak, an SMS blast (one link, tens of thousands of clicks in minutes), and a 2-year projection.
- [ ] Generate realistic test data with a skewed click distribution (20% of links get 80% of clicks).
- [ ] Run each scenario from a load-generator VM inside the lab (not from your laptop over Tailscale) and record throughput, p50/p95/p99 latency, error rate and resource usage.
- [ ] Find the first bottleneck (CPU, DB connections, cache, network) and explain it using metrics and traces.
- [ ] Configure a Horizontal Pod Autoscaler and show it scaling during the peak scenario.
- [ ] Write `docs/05-capacity.md`: the maximum requests per second one replica handles within SLO, headroom at launch scale, and what breaks first on the way to 2-year scale.

**Exit conditions**

- [ ] The SMS-blast scenario meets the redirect SLO.
- [ ] `docs/05-capacity.md` compares measured numbers with the G1 estimates and explains any large differences.
- [ ] HPA scaling is demonstrated with a dashboard screenshot saved in the doc.
- [ ] Any SLO miss has a GitHub issue that carries into G8 or G9.

## G8 — Chaos experiments and postmortems

**Goal:** learn how Linkly actually fails, and whether you can see it fail.

**Entry conditions**

- [ ] G7 is Done, so you have a baseline of normal behavior under load.
- [ ] A chaos tool is installed (Chaos Mesh, or manual `kubectl` and `tc`).

**Tasks**

For each experiment, write the hypothesis *before* running it: what you expect users to see, which alert should fire and how fast. Run it under steady load.

- [ ] Redis goes down.
- [ ] Postgres gets 500 ms of extra latency.
- [ ] Database connections run out.
- [ ] The Postgres primary pod is killed. Does CloudNativePG fail over, and how long are writes unavailable?
- [ ] The queue or worker stops for 30 minutes.
- [ ] A bad release (one returning 5% errors) is merged and deployed by Argo CD.
- [ ] API pods are killed during peak load.
- [ ] One R715 host is powered off. Test once with the host holding one k3s server, and once with the host holding two (the etcd quorum risk).
- [ ] Jenkins is down during an urgent fix. Can you still ship, and how?
- [ ] The malicious-URL check service is unavailable.
- [ ] Disaster recovery: restore Postgres from S3 backups into a rebuilt cluster, and measure how much data was lost and how long it took.
- [ ] After each one, write a blameless postmortem in `docs/postmortems/` covering summary, timeline, impact, detection time, root cause, what went well, what went badly, and action items.

**Exit conditions**

- [ ] At least 8 experiments are complete, each with a postmortem, including the host power-off and the S3 restore.
- [ ] Each postmortem compares the hypothesis with the result and records time to detect.
- [ ] Every action item has a GitHub issue with a priority.
- [ ] Any gap in the design's failure-modes table is fixed in `docs/03-design.md`, with a change log entry.

## G9 — Hardening and v1.0 release

**Goal:** fix what G7 and G8 uncovered, clean up the code and present the project.

**Entry conditions**

- [ ] G8 is Done, and every action item is a prioritized GitHub issue.

**Tasks**

- [ ] Close every high-priority action item. Typical fixes: timeouts and retry budgets, circuit breakers, connection pool tuning, load shedding, canary deploys with Argo Rollouts.
- [ ] Re-run every experiment that failed its hypothesis and record the new result in its postmortem.
- [ ] Do a security pass: dependency scan (pip-audit), static analysis (bandit), container image scan, Jenkins plugin updates, and an OWASP Top 10 checklist against the API.
- [ ] Do a refactoring pass: reread the code as a reviewer, remove duplication, improve names, add missing tests.
- [ ] Optional: deploy the same Helm charts to a short-lived EKS cluster created with Terraform, run a smoke test, and destroy it the same day. Record the cost.
- [ ] Optional: run the Datadog agent alongside Prometheus for a week and write up how the two compare.
- [ ] Make the repository public if it isn't already, and confirm the Jenkins pull request restrictions hold.
- [ ] Finish the README: architecture diagram, SLOs, key results with numbers, links to the design doc, ADRs and postmortems.
- [ ] Write `docs/06-retrospective.md`: what went well, what you'd do differently, and what you learned about SWE and SRE.
- [ ] Draft two or three resume bullets with real numbers from G7 and G8.

* [ ] Do a rolling k3s minor-version upgrade and a Postgres minor upgrade with no redirect downtime, following a runbook you write first.

**Exit conditions**

- [ ] All high-priority issues are closed. Lower-priority ones are labeled for later.
- [ ] Re-run experiments meet their SLOs, or the reason they don't is documented.
- [ ] No critical or high findings from the security scans remain open.
- [ ] `v1.0.0` is tagged with release notes.
- [ ] Someone who has never seen the project can understand it from the README in 10 minutes.

## G10 — Kafka migration

**Goal:** move the click-event pipeline from Redis Streams to Kafka in production without losing or double-counting a single click.

**Entry conditions**

- [ ] G9 is Done and `v1.0.0` is tagged.
- [ ] Current click counts and queue lag are visible on dashboards, so you can compare before and after.

**Tasks**

- [ ] Write an ADR for the migration: why Kafka, the topic design (partition key, partition count, retention), delivery guarantees, and the cutover plan.
- [ ] Deploy Kafka with the Strimzi operator via Argo CD. Spread brokers across both hosts and turn on its metrics in Prometheus.
- [ ] Put the event publisher behind an interface in the service layer, with Redis and Kafka implementations selected by config.
- [ ] Make the worker idempotent (for example, deduplicate on an event ID), so replays and dual delivery can't double-count.
- [ ] Phase 1, dual-write: publish to both Redis and Kafka. Run a Kafka consumer that writes to a shadow table.
- [ ] Phase 2, verify: compare the shadow aggregates with the live ones for several days, including under a k6 load test.
- [ ] Phase 3, cut over: switch the live worker to Kafka, keeping Redis publishing as a fallback for a set period.
- [ ] Phase 4, clean up: stop Redis publishing, remove the old consumer code, update `docs/03-design.md` and the diagrams.
- [ ] Add Kafka failure modes to the failure-modes table, and run two experiments: a broker killed, and consumer lag building up.
- [ ] Write a migration postmortem or retrospective, even if nothing went wrong.

**Exit conditions**

- [ ] Shadow and live aggregates matched within your stated tolerance before cutover, with the comparison saved in the repo.
- [ ] Redirect SLOs were met throughout the migration.
- [ ] Redis Streams code and config are removed.
- [ ] Both Kafka experiments have postmortems.
- [ ] `v1.1.0` is tagged, and the README and design doc describe the new pipeline.

## Session log

Add one row at the end of every work session, newest at the top. "Next step" should be specific enough to start on without thinking, for example "Write NFR-4 to NFR-6 (latency)" rather than "Continue requirements."

| Date | Gate | What I did | Next step | Blockers or notes |
| --- | --- | --- | --- | --- |
| 2026-11-25 | — | Re-estimated in hours for 3 h/day, 6 days a week; added remote lab access risks and recovery tasks (servers are off-site, Tailscale only) | Start G0: create the `linkly` repository | Record actual hours per gate in the tracker |
| 2026-11-25 | — | Confirmed k3s (not kind or kubeadm) for the lab cluster, with Cilium, MetalLB and a standard ingress controller replacing the k3s defaults | Start G0: create the `linkly` repository, then install local tools | kubeadm is a possible follow-up project after Linkly |
| 2026-11-25 | — | Chose the tech stack (Python/FastAPI, Proxmox lab with k3s, Jenkins + Argo CD, AWS within $20, Redis Streams then Kafka). Updated this plan and added G10. | Start G0: create the `linkly` repository, then install local tools | Check whether the GitHub Student Developer Pack offers Datadog credits |
| 2026-11-25 | — | Wrote the product brief and this master plan | Start G0: create the `linkly` repository | — |
