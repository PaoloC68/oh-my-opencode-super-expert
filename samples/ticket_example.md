# Provide Deployment Option for PR-Agent (Automated PR Descriptions) — IT Security Compliant

**Project:** [INTERNAL PROJECT]
**Type:** Consultancy / Assistance on Server
**Priority:** Normal
**Component:** CI/CD Tools
**Linked Ticket:** [PREDECESSOR-TICKET] (predecessor)

---

## TL;DR
"Wir benötigen einen Server/VM auf dem ein Docker-Container dauerhaft betrieben werden darf, mit Netzwerkzugang zu Bitbucket Server und P API. Den Container erstellen und konfigurieren wir selbst."

We need a host (VM, bare-metal, or K8s namespace) where a single Docker container can run persistently. The container runs PR-Agent, an open-source tool that auto-generates pull request descriptions using our internal LLM API. IT Security has approved the approach. No external internet access needed — the container talks to exactly two internal endpoints: Bitbucket Server and the LLM API. Resource footprint is ~200–400 MB RAM. Setup takes 1–2 hours once the host is available.

**What we need from infrastructure:**
1. A host with Docker or Podman runtime
2. Network access from the container to Bitbucket Server and the internal LLM API (HTTPS 443)
3. Inbound access from Bitbucket Server to the container on port 3000
4. Guidance on SSL certificate trust for internal PKI

Everything else (container configuration, startup, testing, webhook setup) is handled by the CI/CD Tools team.

---

## Objective

Provide a deployment target (host, VM, or container runtime) for the PR-Agent service within the internal corporate network. PR-Agent will generate automated pull request descriptions for the main application repository by connecting to an internal OpenAI-compatible LLM API. All processing remains on-premise — no data leaves the corporate network.

---

## Background

Developers creating pull requests in the Bitbucket Server repository frequently submit PRs with empty or incomplete descriptions. This impacts code review quality, traceability, and onboarding of new team members.

PR-Agent (https://github.com/qodo-ai/pr-agent) is an open-source tool that analyzes git diffs and commit messages, sends them to an LLM, and writes a structured PR description back to Bitbucket via the REST API.

**IT Security clearance** has been obtained from the local IT Security team. The internal LLM team has confirmed that the API supports data classified up to the required security level for this use case.

---

## Technical Architecture

### Components and data flow

```
┌──────────────────┐     webhook (pr:opened)     ┌──────────────────┐
│  Bitbucket Server │ ──────────────────────────▸ │    PR-Agent      │
│  (internal        │                             │  (Docker container│
│   instance)       │ ◂────────────────────────── │   webhook server) │
└──────────────────┘   REST API (update PR desc)  └────────┬─────────┘
                                                           │
                                                           │ POST /v1/chat/completions
                                                           ▼
                                                  ┌──────────────────┐
                                                  │  Internal LLM    │
                                                  │  API (OpenAI-    │
                                                  │  compatible)     │
                                                  └──────────────────┘
```

### Network requirements

The PR-Agent container requires outbound access to exactly **two** internal endpoints:

| Destination | Protocol | Port | Purpose |
|---|---|---|---|
| Bitbucket Server instance | HTTPS | 443 | Bitbucket Server REST API (read PR, update description) |
| Internal LLM API endpoint | HTTPS | 443 | OpenAI-compatible LLM API (chat completions) |

**No external internet access is required or desired.** All other outbound connections should be blocked by firewall rules or network policy.

The container exposes **one inbound port** (default: 3000) to receive webhook POST requests from Bitbucket Server.

| Source | Destination | Protocol | Port | Purpose |
|---|---|---|---|---|
| Bitbucket Server instance | PR-Agent container | HTTPS/HTTP | 3000 | Bitbucket webhook delivery |

---

## Software Details

### PR-Agent

- **Repository:** https://github.com/qodo-ai/pr-agent
- **License:** AGPL-3.0 (see Licensing section below)
- **Language:** Python (100%)
- **Docker image:** `codiumai/pr-agent:latest` (official, unmodified)
- **Docker target for Bitbucket Server:** `bitbucket_server_webhook`
- **Image size:** ~500 MB
- **Runtime memory:** ~200–400 MB
- **Startup time:** <10 seconds

### Container build command (for internal registry)

```bash
# Pull official image
docker pull codiumai/pr-agent:latest

# Or build the Bitbucket Server webhook target from source
git clone https://github.com/qodo-ai/pr-agent.git
cd pr-agent
docker build . \
    -t internal-registry.corp/pr-agent:bitbucket_server_webhook \
    --target bitbucket_server_webhook \
    -f docker/Dockerfile
```

### Runtime configuration

PR-Agent requires two configuration files:

**`.secrets.toml`** — credentials (injected via environment variables or mounted file):

```toml
[openai]
key = "<LLM_API_KEY>"
api_base = "https://llm-api.internal.corp/v1"

[bitbucket_server]
bearer_token = "<BITBUCKET_SERVICE_ACCOUNT_PAT>"
```

**`configuration.toml`** — behavior settings:

```toml
[config]
git_provider = "bitbucket_server"
model = "openai/<model-name>"
fallback_models = ["openai/<fallback-model>"]
custom_model_max_tokens = 128000

[bitbucket_server]
url = "https://bitbucket.internal.corp"

[pr_description]
publish_labels = false
add_original_user_description = true
generate_ai_title = false

[pr_reviewer]
automatic_review = false
```

### Container run command

```bash
docker run -d \
    --name pr-agent \
    --restart unless-stopped \
    -p 3000:3000 \
    -e CONFIG.GIT_PROVIDER=bitbucket_server \
    -e OPENAI.KEY=<LLM_API_KEY> \
    -e OPENAI.API_BASE=https://llm-api.internal.corp/v1 \
    -e CONFIG.MODEL=openai/<model-name> \
    -e CONFIG.CUSTOM_MODEL_MAX_TOKENS=128000 \
    -e BITBUCKET_SERVER.URL=https://bitbucket.internal.corp \
    -e BITBUCKET_SERVER.BEARER_TOKEN=<BITBUCKET_PAT> \
    codiumai/pr-agent:bitbucket_server_webhook
```

---

## Source Code Review Findings

A manual review of the PR-Agent source code was conducted with the following findings:

1. **No data exfiltration mechanisms detected.** The code does not contain any hardcoded external URLs, telemetry endpoints, or analytics callbacks. All HTTP calls are directed to user-configured endpoints (LLM API and Git provider).

2. **No volume mounts required.** The container operates statelessly — it receives a webhook, processes it, and responds. No persistent storage, no host filesystem access.

3. **No privileged capabilities required.** The container runs as a standard unprivileged process. No `--privileged` flag, no host networking, no elevated permissions.

4. **Dependencies are standard Python packages.** The `requirements.txt` includes well-known libraries: `fastapi`, `uvicorn`, `httpx`, `pydantic`, `tiktoken`, `litellm`. No obscure or suspicious dependencies.

5. **Transparent request flow.** The webhook handler in `pr_agent/servers/bitbucket_app.py`:
   - Receives the Bitbucket webhook payload (JSON)
   - Extracts PR metadata (project, repo, PR ID)
   - Calls the configured LLM endpoint with the diff content
   - Writes the generated description back via Bitbucket REST API
   - Returns HTTP 200

6. **No credential persistence.** API keys are read from environment variables or mounted config files at startup. They are not written to disk, logged, or transmitted to any endpoint other than the configured services.

---

## Security Posture

### Defense in depth

| Layer | Measure |
|---|---|
| **Network** | Firewall/network policy restricts egress to internal LLM API and Bitbucket Server only. All other outbound blocked. |
| **Application** | Unmodified official Docker image. Source code reviewed — no exfiltration vectors found. |
| **Credentials** | LLM API key and Bitbucket PAT stored in Jenkins Credential Manager or equivalent secure store (HashiCorp Vault, K8s Secrets). Never hardcoded. |
| **Data classification** | Internal LLM API is approved for data up to the required classification level. Source code in the repository is classified at this level or below. |
| **Container isolation** | Unprivileged container, no host mounts, no privileged capabilities, read-only filesystem possible. |
| **Monitoring** | Container logs can be forwarded to existing log aggregation. Health endpoint available for monitoring. |

### IT Security clearance

The local IT Security team has reviewed the use case and confirmed:
- The internal LLM models can be used for this purpose
- The security approach (internal-only networking, no external API calls) is acceptable
- Formal security approval was given and can be documented if required

---

## Licensing: AGPL-3.0 Compliance

PR-Agent is licensed under AGPL-3.0. The compliance approach is as follows:

1. **We use the unmodified official Docker image** (`codiumai/pr-agent:latest`). We do not modify, fork, or redistribute the source code.

2. **AGPL-3.0 permits internal use** without triggering the copyleft distribution clause, as long as the software is not offered as a network service to third parties. PR-Agent is used exclusively within the corporate internal network for internal development — it is not exposed to external users or customers.

3. **No proprietary code is combined with PR-Agent.** Configuration files (`.toml`) are not considered derivative works under AGPL.

4. **Legal review is pending** with the Legal/IPR department. This analysis is provided for technical context and does not constitute legal advice.

---

## Deployment Options

### Recommended: Persistent Container Service (Option B)

Deploy PR-Agent as a long-running container on any available host with Docker or Podman.

**How it works:**
- PR-Agent starts and listens for webhooks on port 3000
- Bitbucket Server webhook sends PR events directly to the container
- PR-Agent processes requests sequentially in its built-in async web server
- No Jenkins involvement at runtime

**Requirements from infrastructure:**
- A host (VM, bare-metal, or K8s namespace) capable of running a Docker/Podman container
- Network access as described in the Network Requirements section
- DNS resolution for the two target endpoints
- SSL certificate trust for internal PKI (if LLM API uses internal CA)

**Advantages:**
- Matches PR-Agent's native architecture (built-in `bitbucket_server_webhook` mode)
- Handles concurrent PRs without race conditions (sequential async processing)
- No Jenkins runtime dependency — direct Bitbucket ↔ PR-Agent interaction
- Low resource footprint (~200–400 MB RAM, minimal CPU)
- Once deployed, requires minimal maintenance

**Estimated effort:** 1–2 hours for initial setup once host is provided.

### Alternative: Jenkins Ephemeral Pipeline (Option A)

Run PR-Agent as an ephemeral container within a Jenkins multi-branch pipeline.

**How it works:**
- Bitbucket webhook triggers a Jenkins pipeline on PR creation
- Jenkins spins up a PR-Agent container, executes description generation
- Container updates PR via Bitbucket REST API, then is destroyed

**Requirements from infrastructure:**
- Docker-enabled Jenkins agents
- Multi-branch pipeline configuration
- Webhook configuration pointing to Jenkins

**Considerations and risks:**
- **Concurrency under load:** Each PR event (open, update, comment) spawns a separate Jenkins job and Docker container. In a realistic scenario — e.g., 2 PRs opened within minutes, each receiving 3 reviewer comments — this produces up to 8 concurrent Jenkins jobs, each pulling and starting a container. On an already utilized Jenkins server, this competes directly with build and test pipelines for executor slots, potentially delaying CI/CD operations across the team.
- **Race conditions on the Bitbucket API:** Multiple ephemeral containers updating the same PR description concurrently will encounter Bitbucket Server's optimistic locking (version field). Without coordinated retry logic — which must be custom-built and maintained — updates will fail with HTTP 409 conflicts.
- **Cascading failures:** If Docker image pulls from the internal registry are slow or the registry is temporarily unavailable, Jenkins jobs queue up. A backlog of queued PR-Agent jobs can exhaust executor capacity, blocking unrelated builds. The failure mode of a lightweight auxiliary service should never impact the core CI/CD pipeline.
- **Maintenance overhead:** The Jenkins multi-branch pipeline configuration (webhook trigger, Docker agent setup, credential injection, error handling, retry logic) becomes an ongoing maintenance responsibility. Any Jenkins upgrade, agent reconfiguration, or plugin change can break the pipeline.
- **Architectural mismatch:** PR-Agent ships with a purpose-built asynchronous webhook server (`bitbucket_server_webhook`) designed for exactly this use case. Running it as ephemeral containers through Jenkins bypasses this built-in capability and replaces it with custom orchestration that must be developed and maintained internally.

---

## Credentials Required

Two credentials need to be created and securely stored:

| Credential | Source | Scope | Action needed |
|---|---|---|---|
| **LLM API Key** | Internal LLM team | Service account for the project | Request API key for the service account |
| **Bitbucket Server PAT** | Bitbucket admin | PR read/write on target repository | Create HTTP Access Token for service account |

Both credentials should be stored in the Jenkins Credential Manager or an equivalent secure store accessible by the deployment environment.

---

## Bitbucket Webhook Configuration

Once the PR-Agent container is running, configure a webhook on the target repository:

1. Navigate to: `Repository Settings → Webhooks → Create webhook`
2. **Name:** `PR-Agent Description Generator`
3. **URL:** `http://<pr-agent-host>:3000/webhook`
4. **Events:** Select `Pull Request: Opened`
5. **Authentication:** None (or HMAC secret if configured in PR-Agent)

---

## Requested Action

**Provide a deployment target** (host/VM/container runtime) where the PR-Agent Docker container can run persistently with the network access described above. Specifically:

1. Identify or provision a host with Docker/Podman runtime
2. Confirm network connectivity to the internal LLM API and Bitbucket Server
3. Confirm inbound access from Bitbucket Server to the container on port 3000
4. Advise on SSL certificate trust requirements for internal PKI

The container deployment itself (configuration, startup, testing) will be handled by the CI/CD Tools team.

---

## Dependencies and Open Items

| Item | Status | Owner |
|---|---|---|
| IT Security clearance | ✅ Approved | IT Security team |
| Internal LLM API access | ✅ Confirmed | LLM platform team |
| AGPL-3.0 license review | ⏳ Pending response | Legal/IPR department |
| LLM API key creation | ⏳ Not yet requested | LLM platform team |
| Bitbucket service account PAT | ⏳ Not yet created | Bitbucket admin |
| **Deployment host/runtime** | ❌ **This ticket** | **Infrastructure** |
