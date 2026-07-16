# PP-SPEC-016: AgenTwin — Configurable Agent Under Test

**Specification ID:** PP-SPEC-016  
**Status:** DRAFT  
**Version:** 0.1.0  
**Date:** 2026-07-16  
**Author:** Nebulonium, Inc. / HACKERverse®  
**License:** CC BY 4.0  
**Repository:** https://github.com/proofprotocol/agentwin-spec  

---

## 1. Abstract

AgenTwin is a configurable, naive MCP client that serves as the Agent Under Test (AUT) in Proof Protocol certification runs. AgenTwin is a behavioral stand-in for any production AI agent — it mimics the tool surface, permissions, and capability profile of a target agent without applying model-level safety filters.

AgenTwin is deliberately naive. It executes tool calls as instructed. It does not refuse. It does not reason about intent. This naivety is a feature, not a bug — it ensures that the Agent Security Control (ASC) and ProofTwin are the only lines of defense being evaluated. The agent itself is not under evaluation for its own safety properties. The system protecting it is.

In production deployments, AgenTwin is replaced by the customer's real agent. The surrounding stack — ProofTwin, the ASC, Arena7, ProofRegister — remains identical. AgenTwin is the removable, replaceable component.

---

## 2. Motivation

A meaningful security evaluation requires a defined target. You cannot evaluate how well a security control protects an agent without specifying what the agent can do, what tools it has access to, and how it behaves under normal conditions.

Existing benchmarks use a trivial stand-in (`cat`, echo servers) that cannot represent real agent behavior. This produces wire-layer results only — the security control either blocked the payload or it didn't. There is no behavioral layer.

AgenTwin fills this gap by providing:

- A configurable agent that faithfully represents a class of production AI agents
- A defined tool surface that the ACR can test against
- A behavioral baseline that ProofTwin can compare against during adversarial sessions
- A swappable component that production agents can replace without changing the surrounding attestation stack

---

## 3. Definitions

All terms defined in PP-SPEC-015 (ProofTwin) and PP-SPEC-017 (PP-MCP) apply. Additional terms:

**AUT (Agent Under Test)** — The agent component being evaluated as part of a System Under Test (SUT). AgenTwin is the reference AUT.

**Tool Profile** — A machine-readable declaration of the tools AgenTwin exposes during a test session. Determines which ACR cases are applicable.

**Behavioral Baseline** — A recorded set of tool calls AgenTwin makes in response to a clean, non-adversarial prompt set. Used by ProofTwin for deviation detection.

**Naivety** — The property of executing tool calls without applying safety filters, refusals, or intent reasoning. Required for valid AUT behavior during certification runs.

**Production Agent** — The real AI agent that AgenTwin stands in for during testing. May be a Claude-based assistant, an autonomous coding agent, a customer service bot, or any MCP-capable agent.

**AgenTwin Profile** — The complete configuration of an AgenTwin instance: tool profile, baseline prompt set, capability claims, and target agent description.

---

## 4. Architecture

### 4.1 Position in Stack

```
ACR (Attack Corpus Runner)
    │  adversarial cases
    ▼
AGENTWIN (this spec — MCP client / Agent Under Test)
    │  MCP tool calls
    ▼
PROOFTWIN (PP-SPEC-015 — attestation layer)
    │  forwards + logs
    ▼
ASC (Agent Security Control — PP-SPEC-017)
    │  block or allow
    ▼
NETWORK / TARGET
```

AgenTwin is the topmost component in the SUT. It initiates all tool calls. It does not receive inbound connections — it connects outbound to ProofTwin.

### 4.2 Relationship to Production Agent

```
TESTING MODE                    PRODUCTION MODE
─────────────────               ─────────────────
AGENTWIN (naive stand-in)  →   REAL AGENT (customer's agent)
    │                               │
PROOFTWIN                      PROOFTWIN
    │                               │
ASC                            ASC
```

The swap is configuration-only. ProofTwin and the ASC do not change. The AgenTwin Profile documents the tool surface and capability claims of the target agent so the substitution is faithful.

---

## 5. Naivety Requirements

AgenTwin MUST satisfy the following naivety requirements during certification runs:

**5.1 No Refusals**  
AgenTwin MUST NOT refuse any tool call based on content, intent, or safety reasoning. If a tool call is in its tool profile, it executes it.

**5.2 No Safety Filters**  
AgenTwin MUST NOT apply content filtering, prompt injection detection, or output sanitization. These are the ASC's responsibilities.

**5.3 No Intent Reasoning**  
AgenTwin MUST NOT evaluate whether a tool call is benign or malicious before executing it. It executes all calls that conform to the tool schema.

**5.4 No Self-Modification**  
AgenTwin MUST NOT modify tool call arguments before passing them to ProofTwin. Arguments are passed as received.

**5.5 Naivety Attestation**  
AgenTwin MUST include a naivety attestation in its session metadata confirming these requirements are met:

```json
{
  "naivety_attestation": {
    "no_refusals": true,
    "no_safety_filters": true,
    "no_intent_reasoning": true,
    "no_self_modification": true,
    "attested_at": "2026-07-16T14:00:00.000Z"
  }
}
```

---

## 6. Tool Profile Schema

The Tool Profile declares the complete tool surface AgenTwin exposes during a test session. The ACR uses this to determine applicable cases. ProofTwin uses this to validate scope.

```json
{
  "schema_version": "HV-ATP-1.0",
  "profile_id": "<uuid>",
  "profile_hash": "<sha256 of this document>",
  "component_id": "agentwin-01",
  "component_type": "agent",
  "version": "1.0.0",
  "target_agent": {
    "description": "Claude-based customer service agent",
    "capabilities": ["web_fetch", "file_read", "tool_call"],
    "deployment_context": "enterprise_saas"
  },
  "tools": [
    {
      "name": "fetch",
      "description": "Fetch a URL and return the response body",
      "inputSchema": {
        "type": "object",
        "properties": {
          "url": { "type": "string" }
        },
        "required": ["url"]
      }
    },
    {
      "name": "read_file",
      "description": "Read a file from the local filesystem",
      "inputSchema": {
        "type": "object",
        "properties": {
          "path": { "type": "string" }
        },
        "required": ["path"]
      }
    },
    {
      "name": "execute_shell",
      "description": "Execute a shell command",
      "inputSchema": {
        "type": "object",
        "properties": {
          "command": { "type": "string" }
        },
        "required": ["command"]
      }
    }
  ],
  "transports": ["mcp_stdio", "mcp_http"],
  "capability_claims": [
    "url_fetch",
    "file_read",
    "shell_execution",
    "mcp_tool_call"
  ]
}
```

---

## 7. Baseline Prompt Set

Before the adversarial session begins, AgenTwin executes a versioned baseline prompt set — a fixed sequence of benign tool calls with known expected behavior. This establishes the Behavioral Baseline that ProofTwin uses for deviation detection.

### 7.1 Baseline Requirements

- Baseline prompts MUST be versioned and content-hashed
- Baseline prompts MUST cover each tool in the Tool Profile at least once
- Baseline prompts MUST be benign — no adversarial content
- Baseline execution MUST occur immediately before the adversarial session with no configuration changes between phases

### 7.2 Baseline Record Format

```json
{
  "baseline_version": "v1.0.0",
  "baseline_hash": "<sha256>",
  "captured_at": "2026-07-16T14:00:30.000Z",
  "tool_calls": [
    {
      "seq": 1,
      "tool_name": "fetch",
      "arguments": { "url": "https://example.com" },
      "response_status": "success",
      "response_body": { "status": 200, "body": "..." },
      "latency_ms": 45
    },
    {
      "seq": 2,
      "tool_name": "read_file",
      "arguments": { "path": "/tmp/baseline-test.txt" },
      "response_status": "success",
      "response_body": { "content": "baseline test content" },
      "latency_ms": 2
    }
  ]
}
```

---

## 8. MCP Client Interface

AgenTwin connects to ProofTwin as an MCP client. The full interface contract is defined in PP-SPEC-017 (PP-MCP). Key requirements:

**8.1 Session Binding**  
AgenTwin MUST include `hv_session_id` in the `_meta` field of every tool call:

```json
{
  "_meta": {
    "hv_session_id": "<session_id>",
    "hv_agent_id": "agentwin-01",
    "hv_agent_version": "1.0.0",
    "hv_mcp_version": "1.0"
  }
}
```

**8.2 Tool Discovery**  
AgenTwin MUST call `tools/list` on ProofTwin at session start and use only the tools returned. It MUST NOT call tools outside the declared tool surface.

**8.3 Error Handling**  
AgenTwin MUST surface all errors to ProofTwin without suppression. If a tool call returns an error, AgenTwin records and forwards the error as-is.

---

## 9. Session Lifecycle

AgenTwin participates in the session lifecycle defined in PP-SPEC-015:

| Phase | AgenTwin Action |
|-------|----------------|
| Phase 0 — Pre-Commitment | Waits for session_id from ProofTwin |
| Phase 1 — Baseline Capture | Executes baseline prompt set, records all tool calls |
| Phase 2 — Adversarial Session | Executes tool calls as directed by ACR cases |
| Phase 3 — Deviation Analysis | Passive — ProofTwin performs analysis |
| Phase 4 — PTBR Emission | Passive — ProofTwin emits the record |

---

## 10. AgenTwin Profile Configuration

AgenTwin is configured via a YAML file:

```yaml
agentwin:
  id: "agentwin-01"
  version: "1.0.0"

target_agent:
  description: "Claude-based enterprise assistant"
  deployment_context: "enterprise_saas"

prooftwin:
  addr: "127.0.0.1:9995"
  transport: "mcp_http"

tool_profile: "./profiles/enterprise-assistant.json"

baseline:
  prompt_set: "./baseline/v1.0.0"
  verify_hash: true

naivety:
  enforce: true
  log_all_calls: true
```

---

## 11. Production Agent Substitution

In production, the real agent replaces AgenTwin. The substitution is valid when:

1. The real agent exposes the same tool surface as the AgenTwin Tool Profile
2. The real agent connects to ProofTwin as an MCP client
3. The real agent includes `hv_session_id` in tool call metadata
4. ProofTwin is notified of the substitution in session metadata

```json
{
  "agent_mode": "production",
  "production_agent": {
    "id": "customer-agent-prod-01",
    "description": "Real production agent",
    "agentwin_profile_id": "<uuid of the AgenTwin profile it replaces>"
  }
}
```

The PTBR produced in production mode is distinguished from test mode by the `agent_mode` field. ProofStamp certification requires at least one test-mode run with AgenTwin before production-mode monitoring is valid.

---

## 12. Reference Implementation

The reference AgenTwin implementation is a Python MCP client. Minimum viable structure:

```
agentwin/
├── client.py        # MCP client — connects to ProofTwin
├── tools.py         # tool executor — naive, no filters
├── baseline.py      # baseline prompt set runner
├── profile.py       # tool profile loader and validator
├── session.py       # session lifecycle management
└── config.yaml      # AgenTwin configuration
```

---

## 13. Relationship to Existing Standards

| Standard | Relationship |
|----------|-------------|
| PP-SPEC-015 ProofTwin | ProofTwin witnesses AgenTwin. AgenTwin is ProofTwin's primary client. |
| PP-SPEC-017 PP-MCP | AgenTwin implements the HV-MCP client interface defined in PP-SPEC-017. |
| PP-SPEC-006 PES | AgenTwin tool calls are the source events from which PES is computed. |
| PP-SPEC-009 ProofStamp | ProofStamp certification requires at least one complete AgenTwin test run per SUT configuration. |
| aeb-gauntlet | AgenTwin replaces `cat` as the agent-side component in aeb-gauntlet compatible runs. |

---

## 14. Prior Art and Timestamps

This specification was authored by Nebulonium, Inc. and anchored to the NIST Randomness Beacon and Zenodo DOI registry on or before 2026-07-25.

---

## 15. Status and Roadmap

| Milestone | Target |
|-----------|--------|
| PP-SPEC-016 v0.1 published to Zenodo | 2026-07-25 |
| Reference implementation (agentwin v0.1) | 2026-08-01 |
| First AgenTwin-based SUT certification run | 2026-08-15 |
| PP-SPEC-016 v1.0 stable | 2026-09-01 |

---

## References

- PP-SPEC-015: ProofTwin — Agent Behavioral Attestation Layer
- PP-SPEC-017: PP-MCP Interface Standard
- PP-SPEC-006: Proof Efficacy Score (PES)
- PP-SPEC-009: ProofStamp Certification Criteria
- MCP Specification: https://modelcontextprotocol.io
- ProofRegister: https://chain.proofregister.com
- NIST Randomness Beacon: https://beacon.nist.gov

---

*© 2026 Nebulonium, Inc. Licensed under CC BY 4.0. AgenTwin, ProofTwin, ProofStamp, ProofRegister, HACKERverse, and Proof Economy are trademarks of Nebulonium, Inc.*
