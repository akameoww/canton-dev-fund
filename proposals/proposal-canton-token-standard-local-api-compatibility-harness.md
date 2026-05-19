# Development Fund Proposal: Canton Token Standard Local API Compatibility & CI Harness

**Author:** NODEJUMPER (https://nodejumper.io/)  
**Status:** Submitted  
**Created:** 2026-05-06  
**Label:** daml-tooling
**Champion:** Canton Foundation

---

## Abstract

This proposal requests **650,000 CC** to build the **Canton Token Standard Local API Compatibility & CI Harness** — an open-source, application-facing local development layer for testing Token Standard / CIP-56-style workflows from backend services, SDKs, wallets, QA pipelines, and examples.

This proposal explicitly acknowledges existing Splice Token Standard test infrastructure. The Splice repository already includes `token-standard/splice-token-standard-test`, a Daml-level Token Standard test harness with `AmuletRegistry`, Token Standard package dependencies, Daml scripts, transfer / DvP scenarios, and disclosure handling inside Daml tests.

This project does **not** rebuild that Daml-level harness.

Instead, it builds the missing application-facing layer around existing Splice Token Standard test assets where possible:

1. a local service/API layer for supported Token Standard-style workflows;
2. a supported local CIP-56 OpenAPI-compatible profile for selected v1 workflows;
3. disclosed-contract handling, including ledger-derived `createdEventBlob`;
4. deterministic local scenarios and CI smoke tests;
5. Java/Spring and TypeScript/NestJS examples;
6. documentation and a supported / simulated / deferred compatibility matrix.

The intended local developer experience is expected to be similar to:

```bash
docker compose up -d
ctsu fixture seed --profile basic-token
ctsu scenario run first-transfer --from Alice --to Bob --symbol tUSD --amount 100
ctsu assert holdings --party Bob --symbol tUSD --amount 100
ctsu fixture reset
```

These commands are illustrative examples of the intended local developer ergonomics, not a finalized CLI contract. Exact command names, argument shapes, and supported local workflow boundaries will be finalized during Milestone 1 as part of the architecture, scope, and compatibility work.

The goal is to turn existing Token Standard test assets into a reusable application-facing workflow target that SDK, wallet, backend, QA, and DevRel teams can consume without having to work directly inside Daml test internals.

All software artifacts will be released under the **Apache 2.0** open-source license.

---

## Why NODEJUMPER

NODEJUMPER is a Proof-of-Stake validator and blockchain infrastructure provider active since early 2022. We run secure, self-managed infrastructure with 24/7 monitoring, high availability standards, incident response processes, and a security-first operational approach.

Our team currently supports services across **20+ networks** and has built practical experience in validator operations, infrastructure monitoring, automation, reliable deployment workflows, open-source tooling, onboarding guides, community resources, and developer-facing infrastructure.

NODEJUMPER has maintained a strong operational record, including consistently high uptime and no slashing events across its validator operations.

This proposal is aligned with what we already do well: building reliable, practical infrastructure that lowers barriers for builders and operators. The project is shared developer infrastructure that requires operational discipline, reproducible environments, deterministic CI flows, clear documentation, and long-term maintainability.

For Canton, NODEJUMPER intends to contribute beyond validator operation by building useful open-source tooling that helps application teams, wallet builders, SDK maintainers, QA engineers, and DevRel teams test Token Standard workflows locally before moving to live environments.

---

## Motivation

Canton’s Token Standard creates a common foundation for tokenized assets, wallets, custody providers, and applications. This is critical for interoperability, but it also creates a practical developer experience challenge: before a team can build a useful application flow, it often needs a working token environment.

For many application and SDK teams, the first meaningful milestone is not a MainNet deployment. It is the ability to repeatedly run a local workflow:

1. seed or create a test instrument;
2. seed or mint balances for local parties;
3. retrieve holdings;
4. run a transfer scenario;
5. provide required disclosed contracts;
6. assert expected balances;
7. reset fixture state;
8. run the same workflow automatically in CI.

The existing `splice-token-standard-test` package is useful at the Daml/package level. Application teams usually operate one layer higher: through APIs, SDKs, backend services, CI workflows, and examples.

This project provides that application-facing layer.

---

## Existing Work and Project Delta

The Canton / Splice ecosystem already includes `token-standard/splice-token-standard-test`.

That existing package provides a Daml-level Token Standard test harness, including:

- `AmuletRegistry`;
- Token Standard package dependencies;
- Daml scripts and test utilities;
- transfer and DvP test scenarios;
- Daml-side disclosure handling.

This proposal does **not** replace or duplicate that work.

The proposed project builds the missing application-facing layer around existing Splice Token Standard test assets where possible.

### What this project delivers beyond the existing harness

| Existing `splice-token-standard-test` | This proposal |
| --- | --- |
| Daml-level test harness and scripts | Application-facing local service/API layer |
| Daml utilities such as registry/client helpers | Supported local CIP-56 OpenAPI-compatible profile for selected workflows |
| Disclosures handled inside Daml tests | Explicit disclosed-contract responses for applications, including ledger-derived `createdEventBlob` |
| Daml scripts for test execution | CI-ready scenarios and GitHub Actions examples |
| Useful for Daml/Splice engineers | Useful for backend, wallet, SDK, QA, and DevRel teams |
| Internal package-level test workflows | External examples, compatibility matrix, and local integration documentation |

The project will prefer reuse of upstream Splice Token Standard test packages over rebuilding equivalent Daml logic.

---

## Specification

### 1. Objective

The objective is to deliver an application-facing local compatibility and CI layer for Token Standard / CIP-56-style workflows.

A successful v1 release should allow an external developer or reviewer to:

- clone the repository;
- start the local environment;
- seed or reuse existing Splice Token Standard test assets;
- expose a local API profile for supported workflows;
- retrieve holdings;
- prepare a transfer-like workflow;
- receive required disclosed contracts for supported flows;
- assert expected holdings;
- reset fixture state;
- run the same flow in CI;
- understand which behavior is supported, simulated, or deferred.

### Success looks like

- Developers can complete a first local Token Standard-style transfer workflow from a fresh checkout.
- Application teams can use a local API rather than interacting directly with Daml scripts.
- SDK and wallet teams can test examples against deterministic local token fixtures.
- Supported workflows return `disclosedContracts` with ledger-derived `createdEventBlob` where required.
- CI examples can run fixture seed → scenario execution → holdings assertion → fixture reset.
- Documentation clearly separates local fixture behavior from production Utility infrastructure.
- The repository provides a versioned v1.0.0 release with examples, changelog, compatibility matrix, and maintenance notes.

---

## Implementation Mechanics

This proposal covers four workstreams.

### Workstream A — Existing Splice Test Asset Reuse

**What it is**

The project will start by analyzing and reusing `splice-token-standard-test` where possible. The goal is not to recreate a Daml-level Token Standard harness.

This workstream will define:

- which existing Splice test packages are reused directly;
- which local fixture scenarios are supported in v1;
- which flows require wrappers or additional local API logic;
- which flows are out of scope or deferred;
- how the local service maps fixture-created contracts to API-facing responses.

**Why it matters**

This directly addresses duplication concerns. The project’s value is not rebuilding Token Standard tests; it is making the existing test foundation usable from application-facing workflows.

---

### Workstream B — Local CIP-56 API Compatibility Profile

**What it is**

The local service will expose a supported subset of CIP-56 OpenAPI-compatible endpoints for selected workflows.

The service will not present arbitrary custom `/holdings` or `/transfers` endpoints as canonical Token Standard APIs. If local convenience endpoints are provided for CLI ergonomics, they will be explicitly marked as local-only and non-canonical.

The v1 API profile will focus on a narrow set of supported flows, such as:

- querying local holdings;
- preparing a supported transfer instruction flow;
- returning required disclosed contracts for supported flows;
- exposing fixture status and compatibility metadata;
- resetting or seeding deterministic local fixture state.

**Why it matters**

Application, wallet, and SDK teams need an API surface they can test against. A Daml script alone is not enough for teams building application integrations.

---

### Workstream C — Disclosed Contracts and createdEventBlob Handling

**What it is**

CIP-56 off-ledger transfer workflows require `disclosedContracts` that include `createdEventBlob`. These are Canton-produced serialized contract blobs required by the participant node to verify disclosed contracts when exercising choices.

The service will not fake, manually serialize, or invent `createdEventBlob`.

For supported local workflows, the service will obtain ledger-produced blobs from the local ledger / participant using supported local access paths such as:

- JSON Ledger API active-contract queries with `includeCreatedEventBlob=true`, where available;
- gRPC Ledger API active contract / created event access where applicable;
- existing disclosure helpers from Splice test assets where they can be reused safely.

The service will cache the relevant data by contract id:

- contract id;
- template/interface identifier;
- party visibility context;
- ledger-derived `createdEventBlob`;
- fixture scenario association.

When a supported CIP-56-style response requires disclosed contracts, the service will return the ledger-derived `createdEventBlob` for the relevant local fixture contracts.

Workflows outside the supported v1 local profile will be explicitly listed in the compatibility matrix as deferred or out of scope rather than presented as spec-compliant.

**Why it matters**

This is the key technical difference between a useful compatibility harness and a mock. A spec-aware local profile must preserve the disclosed-contract mechanics that real applications need.

---

### Workstream D — Thin CLI, CI Harness, Examples, and Documentation

**What it is**

The project will provide a thin CLI wrapper and reusable CI examples around the local service.

Illustrative commands may include:

```bash
docker compose up -d
ctsu fixture seed --profile basic-token
ctsu scenario run first-transfer --from Alice --to Bob --symbol tUSD --amount 100
ctsu assert holdings --party Bob --symbol tUSD --amount 100
ctsu fixture reset
```

The CLI will be scriptable, deterministic, and CI-friendly. It will not attempt to become a general-purpose Canton developer CLI.

Deliverables include:

- GitHub Actions smoke-test workflow;
- Java/Spring example;
- TypeScript/NestJS example;
- quickstart guide;
- troubleshooting guide;
- supported / simulated / deferred compatibility matrix;
- v1 release notes and changelog.

**Why it matters**

The output should be usable by application developers and CI systems, not only by Daml engineers running scripts manually.

---

## Architectural Alignment

This project aligns with Canton architecture and ecosystem priorities by strengthening the application development layer around Token Standard workflows.

- **Token Standard adoption:** provides local workflows that help teams understand Token Standard behavior before live deployments.
- **Developer experience:** reduces time to first working token scenario.
- **Interoperability:** gives SDKs, wallets, and applications a shared local test target.
- **CI reproducibility:** enables token workflow testing independent of live-network availability.
- **Operational clarity:** publishes explicit supported, simulated, and deferred behavior so local testing is not confused with production Utility infrastructure.

This project is intentionally conservative. It does not claim production parity with live Utility Registry behavior. It provides a documented local development profile for early integration, examples, and automated testing.

---

## Scope Boundaries and Non-Overlap

This proposal is designed to complement existing and proposed ecosystem tooling.

It is **not**:

- a new Daml-level Token Standard test harness;
- a replacement for `splice-token-standard-test`;
- a new Canton SDK;
- a wallet SDK;
- a browser dApp SDK;
- a wallet discovery layer;
- a general-purpose Canton developer CLI;
- a project scaffolding tool;
- a DAML build/test/deploy lifecycle tool;
- a LocalNet manager;
- a token faucet product;
- a production Utility Registry replacement;
- a general-purpose Token Standard Registry API client;
- a typed workflow SDK or command builder;
- a general-purpose indexer;
- a Token Standard V2 implementation;
- a rewards or traffic analytics tool;
- an auth harness;
- a JWT/OIDC test-token generator;
- a formal verification framework;
- a MainNet deployment project.

It is an application-facing local compatibility and CI harness for selected Token Standard-style workflows.

| Area | Adjacent ecosystem work | Non-overlap position |
| --- | --- | --- |
| Existing Token Standard tests | `splice-token-standard-test` | This project does not replace the Daml-level harness. It reuses or wraps existing test assets where possible and adds an application-facing API/CI layer. |
| Local environment tooling | DevKit, Cantool, cn-quickstart-style environments | This project does not manage LocalNet lifecycle. It provides Token Standard-style fixture scenarios and CI assertions that can run against an existing local environment. |
| General developer CLI | Cantool, dpm-related tooling | This project does not provide project scaffolding, build/test/deploy lifecycle commands, MCP, or plugin infrastructure. The CLI is only a thin wrapper around token fixture workflows. |
| SDKs and typed tooling | Go/Python/.NET/TypeScript SDKs, codegen, app SDKs | This project does not build another SDK. It provides local token scenarios and API-compatible workflows that SDKs and applications can test against. |
| Registry / Utility infrastructure | Utility Registry and live environment services | This project is not a production registry. It provides local-only compatibility and documents supported, simulated, and deferred behavior. |
| Auth and verification tooling | Auth harnesses, JWT/OIDC tools, formal verification frameworks | This project does not provide auth, identity, model checking, trace generation, or formal verification. |
| Rewards / traffic tooling | Traffic-based rewards and app-provider self-check tooling | This project does not analyze rewards, traffic, or Featured App/App Provider economics. |

---

## Required Access and Dependencies

The baseline implementation does not require privileged MainNet, TestNet, or DevNet access. It is designed to run against a local Canton-compatible environment.

Baseline dependencies:

- existing Splice Token Standard test assets where reusable;
- local Canton or compatible local ledger environment;
- Daml tooling for package development and integration;
- Java/Spring Boot for the local service;
- Node.js/TypeScript for the thin CLI wrapper;
- Docker Compose for reproducible local setup;
- GitHub Actions or equivalent CI for smoke tests.

Optional dependencies for later compatibility testing:

- access to CN Quickstart artifacts, if the team chooses to align a reference example with that setup;
- Utility Registry DARs, if available for compatibility-mode experiments;
- DevNet/TestNet validator access, if available, for non-blocking smoke tests.

Live-network smoke tests are not required for v1 acceptance. If access is available, they will be treated as an optional validation profile.

---

## Milestones and Deliverables

### Milestone 1: Existing Work Integration and First createdEventBlob Extraction Path

- **Payment:** 120,000 CC
- **Estimated Delivery:** 1 month after approval
- **Focus:** Establish the exact project delta over `splice-token-standard-test`, define reuse boundaries, and implement the first ledger-derived `createdEventBlob` extraction path for a supported local workflow.

**Deliverables / Value Metrics:**

- Public repository under Apache 2.0 license.
- Existing work analysis documenting how `splice-token-standard-test` is reused.
- Supported / simulated / deferred compatibility matrix draft.
- Initial local skeleton.
- Basic CI that builds the initial project components.
- Initial architecture document.
- First implemented path for retrieving ledger-produced `createdEventBlob` from the local ledger / participant for at least one fixture-created contract.
- Initial disclosed-contract response shape for one supported local CIP-56-style workflow.

### Milestone 2: Local CIP-56 API Compatibility Profile

- **Payment:** 180,000 CC
- **Estimated Delivery:** 2 months after approval
- **Focus:** Implement the application-facing local API profile for supported workflows.

**Deliverables / Value Metrics:**

- Supported local CIP-56 OpenAPI-compatible profile for selected workflows.
- Holdings query flow.
- Supported transfer preparation flow.
- Disclosed-contract response structure.
- Clear marking of local-only convenience endpoints, if any.
- Structured error model.
- OpenAPI documentation for the local profile.

### Milestone 3: createdEventBlob Handling and CI Harness

- **Payment:** 155,000 CC
- **Estimated Delivery:** 3 months after approval
- **Focus:** Implement disclosed-contract handling and deterministic CI workflows.

**Deliverables / Value Metrics:**

- Ledger-derived `createdEventBlob` retrieval and cache for supported fixture-created contracts.
- Disclosed-contract handling for supported v1 workflows.
- Fixture seed and reset capability.
- GitHub Action or reusable CI workflow.
- Compose-based smoke test.
- CI artifacts/logs for debugging failed token flows.

### Milestone 4: Examples, Documentation, and Developer Packaging

- **Payment:** 110,000 CC
- **Estimated Delivery:** 3.5 months after approval
- **Focus:** Package the utility for external developer usage.

**Deliverables / Value Metrics:**

- Thin TypeScript CLI wrapper with documented commands.
- Java/Spring reference example.
- TypeScript/NestJS reference example.
- Quickstart guide.
- Troubleshooting guide.
- Demo walkthrough.
- Release packaging for Docker image, Java service artifact, and CLI package.

### Milestone 5: External Validation, Hardening, and v1 Release

- **Payment:** 85,000 CC
- **Estimated Delivery:** 4 months after approval
- **Focus:** Validate the project with external users, harden the release, and define maintenance.

**Deliverables / Value Metrics:**

- External validation checklist.
- Feedback round with at least two external reviewers or teams.
- Final v1 compatibility matrix.
- Final release notes and changelog.
- 90-day post-release maintenance plan.
- Roadmap for future DevNet/TestNet compatibility mode.

---

## Acceptance Criteria

The Tech & Ops Committee can evaluate milestone completion based on the following evidence:

- **M1:** The proposal repository is public; the project documents how it reuses `splice-token-standard-test`; the compatibility matrix draft is published; the initial local service can retrieve ledger-produced `createdEventBlob` for at least one fixture-created contract through local ledger access; the initial disclosed-contract response shape is documented; CI builds the initial components.
- **M2:** The local API profile supports selected CIP-56-style workflows; OpenAPI documentation is published; local-only convenience endpoints are clearly marked as non-canonical; holdings and transfer preparation flows are demonstrated from clean local state.
- **M3:** Supported flows return ledger-derived `createdEventBlob` inside disclosed-contract responses where required; GitHub Actions runs a full local scenario; fixture reset produces deterministic state; CI logs are useful for debugging.
- **M4:** A new developer can complete the first local token workflow from documentation; Java/Spring and TypeScript/NestJS examples run against the local service; release artifacts are published with versioned tags.
- **M5:** At least two external reviewers or teams complete the validation checklist or provide structured feedback; v1.0.0 release tag is published; final compatibility matrix and maintenance plan are published.

---

## Funding

This request is for a grant of **650,000 Canton Coin (CC)**.

| Milestone | Description | Payment | % of Total |
| --- | --- | ---: | ---: |
| M1 | Existing work integration and first createdEventBlob extraction path | 120,000 CC | 18.46% |
| M2 | Local CIP-56 API compatibility profile | 180,000 CC | 27.69% |
| M3 | createdEventBlob handling and CI harness | 155,000 CC | 23.85% |
| M4 | Examples, documentation, and developer packaging | 110,000 CC | 16.92% |
| M5 | External validation, hardening, and v1 release | 85,000 CC | 13.08% |
| **Total** |  | **650,000 CC** | **100%** |

This funding structure is milestone-driven and forward-looking. No retroactive funding is requested. Each milestone has observable deliverables and acceptance criteria.

Payments are expected after milestone completion and acceptance by the Tech & Ops Committee.

### Funding Rationale

- **M1** funds the implementation foundation that makes the rest of the project credible: reuse of existing Splice test assets and the first ledger-derived `createdEventBlob` extraction path.
- **M2** funds the application-facing API layer that existing Daml scripts do not provide directly.
- **M3** funds the disclosed-contract handling and CI repeatability that make the tool useful for application and SDK teams.
- **M4** funds adoption readiness through examples, packaging, and documentation.
- **M5** funds external validation, release quality, and post-release maintenance planning.

### Cost Basis

The requested amount covers implementation, integration testing, documentation, release packaging, external validation coordination, and a short post-release maintenance period.

Estimated effort:

| Role | Responsibility | Estimated Effort |
| --- | --- | ---: |
| Technical Lead / Daml Architect | Token Standard workflow model, Splice test asset reuse, createdEventBlob architecture | 2.0 person-months |
| Senior Java / Spring Boot Engineer | Local service, OpenAPI profile, ledger integration, integration tests | 2.5 person-months |
| TypeScript / DevEx Engineer | Thin CLI wrapper, GitHub Action, examples, developer workflow | 1.5 person-months |
| QA / CI Engineer | CI harness, smoke tests, reproducibility, release validation | 1.5 person-months |
| Documentation / DevRel Support | Quickstart, examples, troubleshooting, external validation coordination | 0.75 person-months |
| Project / Release Coordination | Planning, changelog, issue triage, milestone reporting | 0.75 person-months |

**Estimated total effort:** approximately 9.0 person-months.

The funding request also covers integration risk, compatibility documentation, release engineering, and 90 days of post-v1 issue triage.

---

## Post-Release Maintenance Commitment

No recurring maintenance funding or hosted-service budget is requested in this proposal.

After v1 release, NODEJUMPER will provide 90 days of post-release issue triage, compatibility clarification, and minor fix support for documented flows. The repository will publish a maintenance plan covering issue labels, support expectations, version compatibility notes, and release/changelog practices.

---

## Co-Marketing

Upon milestone completion and v1 release, the team will coordinate with the Foundation and interested ecosystem contributors on:

- release announcement;
- technical walkthrough;
- demo video;
- quickstart article;
- example integration repositories;
- feedback session with application and SDK teams.

---

## Ecosystem Value

These deliverables are valuable to the Canton ecosystem because they:

- make existing Token Standard test assets easier to consume from applications;
- reduce onboarding time for teams building token-based integrations;
- give SDKs and wallets a reusable local workflow target;
- improve CI reproducibility for Token Standard-style scenarios;
- reduce duplicated app-facing fixture work across teams;
- make Token Standard workflows easier to document, test, and teach.

In practice, this infrastructure helps builders move faster by providing an application-facing local path before live-network dependencies are required.

---

## Rationale

Funding this local API compatibility and CI harness is an efficient way to support Token Standard adoption without duplicating existing Daml test harnesses, SDKs, registry infrastructure, wallet tooling, DevKit work, or general Canton CLI efforts.

The project is intentionally narrow in surface area and broad in utility: it provides API-compatible local workflows, disclosed-contract handling, examples, and CI scenarios that other ecosystem tools can use.

The most important change from the earlier proposal is that this project now treats `splice-token-standard-test` as upstream existing work and builds around it rather than replacing it.

After the v1 release and 90-day post-release maintenance period, any further expansion should be evaluated as a continuation or renewal grant based on demonstrated usage, issue volume, requested compatibility updates, and adoption by application or SDK teams.

---

## References

- `splice-token-standard-test` daml.yaml: https://github.com/hyperledger-labs/splice/blob/main/token-standard/splice-token-standard-test/daml.yaml
- Token transfer instruction OpenAPI: https://github.com/hyperledger-labs/splice/blob/main/token-standard/splice-api-token-transfer-instruction-v1/openapi/transfer-instruction-v1.yaml
- Canton Development Fund README and proposal requirements: https://github.com/canton-foundation/canton-dev-fund
- CIP-0056 Token Standard: https://github.com/canton-foundation/cips/blob/main/cip-0056/cip-0056.md
- Digital Asset Token Standard Integration documentation: https://docs.digitalasset.com/utilities/devnet/overview/registry-user-guide/token-standard.html
