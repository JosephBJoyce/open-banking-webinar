# Open Banking API — AI-Ready, PSD2-Compliant

> **Webinar:** [How to Build an AI-Ready Open Banking API](https://smartbear.com/resources/webinars/how-to-build-an-ai-ready-open-banking-api/)
> **Hosted by SmartBear**

---

## Why this matters: PSD2 today, PSD3 tomorrow

With PSD3 expected to come into force in 2027, now is the time to ensure your API foundations are built to meet both today's requirements and tomorrow's regulation. For European banks, the pressure to stay competitive and innovate has never been greater — and API-powered digital workflows are at the heart of that transformation.

This repository is the live demo for the webinar. It shows how Swagger helps European financial institutions **design compliant APIs**, **prevent integration drift**, and **enable AI-ready banking workflows**. You will see how to design APIs that are easier to test, safer to evolve, and ready for the next generation of fintech and AI consumers.

---

## What you will learn

| Learning objective | Tool / capability shown |
|---|---|
| Enforce PSD2 API guidelines from the first line of design | Swagger Studio AI-generation + governance ruleset |
| Automate regression coverage from design to deployment | Swagger Functional Testing in CI |
| Detect and prevent spec drift by verifying your live API against its OpenAPI document | Bi-Directional Contract Testing (BDCT) via PactFlow |
| Bring your entire API catalogue into AI-native developer tooling | SmartBear MCP Server — standardization gate + portal publishing |

---

## Who this is for

- **Software developers** writing and maintaining APIs for Open Banking or PSD2-compliant workflows
- **API architects** governing API contracts and preventing spec drift in regulated environments
- **Software engineers** exploring how AI-native tooling can accelerate Open Banking delivery

---

## The demo workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│  1. DESIGN                                                              │
│     Swagger Studio → AI-generate a PSD2 Open Banking API               │
│     → Governance checks enforce Berlin Group / PSD2 ruleset in the UI  │
└──────────────────────────────────┬──────────────────────────────────────┘
                                   │ Native GitHub sync (Studio → repo)
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  2. VERIFY CONTRACTS  (workflow 1)                                      │
│     OpenAPI spec published to PactFlow as a BDCT provider contract      │
│     PactFlow cross-checks all consumer pacts against the spec           │
│     → Breaks the build if the spec removes anything a consumer needs    │
└──────────────────────────────────┬──────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  3. FUNCTIONAL TESTS  (workflow 2)                                      │
│     Swagger Functional Testing suites run against sandbox     │
│     Covers: AIS (accounts + transactions), PIS (payments), Consents     │
│     → Regression failures block the deployment pipeline                 │
└──────────────────────────────────┬──────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  4. DEPLOY WITH AI GATES  (workflow 3)                                  │
│                                                                         │
│     Gate 1 — can-i-deploy                                               │
│       PactFlow confirms cross-contract compatibility with all consumers │
│                                                                         │
│     Gate 2 — Standardization scan (SmartBear MCP Server)               │
│       Claude + swagger_scan_api_standardization tool checks the spec    │
│       against the bank's PSD2 / Open Banking governance ruleset         │
│       → CRITICAL or ERROR violations block the deployment               │
│                                                                         │
│     Deploy → record-deployment in PactFlow                              │
└──────────────────────────────────┬──────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  5. PUBLISH DEVELOPER PORTAL  (workflow 4)                              │
│     SmartBear MCP Server → publish_portal_product                       │
│     API reference + changelog updated automatically after each deploy   │
│     → External developers always see docs that match production         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Repository structure

```
openapi/
  open-banking-api.yml              # PSD2 OpenAPI spec — synced from Swagger Studio

.github/workflows/
  1-publish-provider-contract.yml  # BDCT: publish spec to PactFlow on every push
  2-functional-tests.yml           # Reflect: run AIS / PIS / Consent test suites
  3-deploy.yml                     # Deploy with can-i-deploy + standardization gates
  4-publish-portal-docs.yml        # Publish Swagger Portal docs after each deployment
```

---

## Setting up this repo

### Required secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Type | Name | Description |
|---|---|---|
| **Secret** | `PACT_BROKER_TOKEN` | PactFlow API token |
| **Secret** | `REFLECT_API_KEY` | Swagger Functional Testing API key |
| **Secret** | `ANTHROPIC_API_KEY` | Anthropic API key (for Claude Code + MCP gates) |
| **Secret** | `SMARTBEAR_API_KEY` | SmartBear API key (for MCP server tools) |

### Required variables

| Type | Name | Description |
|---|---|---|
| **Variable** | `PACT_BROKER_BASE_URL` | Your PactFlow workspace URL, e.g. `https://yourbank.pactflow.io` |
| **Variable** | `REFLECT_SUITE_AIS` | Reflect suite ID for Account Information tests |
| **Variable** | `REFLECT_SUITE_PIS` | Reflect suite ID for Payment Initiation tests |
| **Variable** | `REFLECT_SUITE_CONSENTS` | Reflect suite ID for Consent Management tests |
| **Variable** | `SWAGGER_PORTAL_PRODUCT_ID` | Portal product ID from Swagger Studio |

### Starting a PactFlow trial

1. Go to [pactflow.io](https://pactflow.io) and start a free trial
2. Create an API token: **Settings → API Tokens → Create token**
3. Add the token and workspace URL as described above
4. Run the workflows in order: `1` → `2` → `3` (workflow `4` is triggered automatically by `3`)

---

## The API: Open Banking / PSD2

The spec in [`openapi/open-banking-api.yml`](openapi/open-banking-api.yml) implements a PSD2-compliant Open Banking API aligned to the **Berlin Group NextGenPSD2** framework. It covers the three core Open Banking service domains:

| Service | Endpoints |
|---|---|
| **Account Information (AIS)** | `GET /accounts`, `GET /accounts/{id}`, `GET /accounts/{id}/transactions` |
| **Payment Initiation (PIS)** | `POST /payments/sepa-credit-transfers`, `GET /payments/sepa-credit-transfers/{id}` |
| **Consent Management** | `POST /consents`, `GET /consents/{id}`, `DELETE /consents/{id}` |

Key design decisions that reflect PSD2/Open Banking requirements:
- `Consent-ID` header required on all AIS and PIS calls (TPP consent scoping)
- `X-Request-ID` UUID header on every request (idempotency + auditability)
- `PSU-IP-Address` required for payment initiation and consent creation (SCA context)
- ISO 20022 transaction status codes (`RCVD`, `ACSC`, `RJCT`, etc.)
- `_links.scaRedirect` in payment and consent responses (SCA redirect flow)

---

## Key webinar takeaways

### 1. Design once, govern always

AI-generation in Swagger Studio dramatically accelerates the first draft — but the real value is the governance layer. Linting rules enforced at design time (before a line of code is written) are orders of magnitude cheaper than finding a non-compliant field after you have published the spec to your TPP partners.

### 2. Spec drift is a regulated-industry risk

In PSD2/Open Banking, your OpenAPI spec is a contractual document shared with licensed TPPs. If your implementation drifts from the spec, you are in breach of contract — and potentially of your regulatory licence conditions. Bi-Directional Contract Testing closes this loop automatically: every push proves the spec and the implementation still agree.

### 3. can-i-deploy is your deployment safety gate

Consumer applications (fintech apps, AISP clients) depend on specific fields and status codes. `can-i-deploy` asks PactFlow: "does any registered consumer break if we deploy this version?" If the answer is yes, the deployment stops — no matter who merged it or when.

### 4. AI-native tooling means Claude can operate your API platform

With the SmartBear MCP Server, Claude Code can call `swagger_scan_api_standardization` as a CI gate, and `publish_portal_product` as an automated post-deploy step. This is what "AI-ready" means for API infrastructure: not just AI-generated specs, but AI agents that can participate in the full API lifecycle.

---

## Resources

- [SmartBear webinar landing page](https://smartbear.com/resources/webinars/how-to-build-an-ai-ready-open-banking-api/)
- [SmartBear MCP Server — Swagger Studio integration](https://developer.smartbear.com/smartbear-mcp/docs/swagger-studio-integration)
- [SmartBear MCP Server — Swagger Portal integration](https://developer.smartbear.com/smartbear-mcp/docs/swagger-portal-integration)
- [Swagger Functional Testing CI/CD docs](https://support.smartbear.com/swagger/functional-testing/docs/en/automation/continuous-integration--ci-cd-.html)
- [PactFlow — Bi-Directional Contract Testing](https://pactflow.io/blog/bi-directional-contract-testing/)
- [Berlin Group NextGenPSD2 framework](https://www.berlin-group.org/nextgenpsd2-downloads)
