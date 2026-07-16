> **Zenodo DOI:** Pending — Target publication 2026-07-25

# PP-SPEC-016 · AgenTwin — Configurable Agent Under Test

**Document ID:** PP-SPEC-016  
**Version:** 0.1 - Draft  
**Status:** Draft  
**License:** CC BY 4.0  
**Maintained by:** Proof Economy™ Standards Alliance (PESA)  
**Repository:** https://github.com/proofprotocol/agentwin-spec  
**Published:** 2026-07-16  

---

## Abstract

AgenTwin is a configurable, naive MCP client that serves as the Agent Under Test (AUT) in Proof Protocol certification runs. AgenTwin is a behavioral stand-in for any production AI agent — it mimics the tool surface, permissions, and capability profile of a target agent without applying model-level safety filters.

AgenTwin is deliberately naive. It executes tool calls as instructed. It does not refuse. It does not reason about intent. This naivety ensures that the Agent Security Control (ASC) and ProofTwin are the only lines of defense being evaluated.

In production deployments, AgenTwin is replaced by the customer's real agent. The surrounding stack — ProofTwin, the ASC, Arena7, ProofRegister — remains identical. AgenTwin is the removable, replaceable component.

---

## Status of This Document

Draft. Subject to change before v1.0.

---

## Table of Contents

1. [Abstract](#abstract)
2. [Motivation](#2-motivation)
3. [Definitions](#3-definitions)
4. [Architecture](#4-architecture)
5. [Naivety Requirements](#5-naivety-requirements)
6. [Tool Profile Schema](#6-tool-profile-schema)
7. [Baseline Prompt Set](#7-baseline-prompt-set)
8. [MCP Client Interface](#8-mcp-client-interface)
9. [Session Lifecycle](#9-session-lifecycle)
10. [Configuration](#10-agentwin-profile-configuration)
11. [Production Agent Substitution](#11-production-agent-substitution)
12. [Reference Implementation](#12-reference-implementation)
13. [Relationship to Existing Standards](#13-relationship-to-existing-standards)
14. [Prior Art and Timestamps](#14-prior-art-and-timestamps)
15. [Status and Roadmap](#15-status-and-roadmap)

---

## Related Specifications

- [PP-SPEC-001](https://github.com/proofprotocol/Defensible-Knowledge-Proof) — Proof Protocol Declaration
- [PP-SPEC-006](https://github.com/proofprotocol/pes-spec) — Proof Efficacy Score
- [PP-SPEC-009](https://github.com/proofprotocol/proofstamp-criteria) — ProofStamp Certification Criteria
- [PP-SPEC-015](https://github.com/proofprotocol/prooftwin-spec) — ProofTwin Behavioral Attestation Layer
- [PP-SPEC-017](https://github.com/proofprotocol/pp-mcp-spec) — PP-MCP Interface Standard

---

## Authors

Nebulonium, Inc. / HACKERverse®  
Craig Ellrod, Founder & CEO  

---

*© 2026 Nebulonium, Inc. Licensed under CC BY 4.0. AgenTwin, ProofTwin, ProofStamp, ProofRegister, HACKERverse, and Proof Economy are trademarks of Nebulonium, Inc.*
