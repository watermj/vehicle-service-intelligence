# Agentic Diagnostic Assistant — Implementation Framework

**Status:** Framework published; implementation in progress.

This extension adds an agentic diagnostic layer to Vehicle Service Intelligence. It is intentionally scoped as a prototype and does **not** claim production readiness.

## Objective

Demonstrate how an LLM-based diagnostic assistant can use structured tools to move from a Vehicle Service Intelligence evidence state to a technician-facing next-best action.

The agent should not invent a diagnosis when evidence is insufficient. It should request the minimum additional evidence needed, reassess confidence, and keep consequential service actions human-approved.

## Reference Workflow

```text
Vehicle Service Intelligence case
        ↓
Diagnostic agent
        ↓
get_vehicle_state()
        ↓
get_fault_codes()
        ↓
get_service_history()
        ↓
Evidence sufficient?
   ┌────┴────┐
   │         │
  NO        YES
   │         │
request_targeted_telemetry()
   │         │
   └────┬────┘
        ↓
Reassess diagnostic confidence
        ↓
Recommend next-best action
        ↓
Technician review / approval
        ↓
Verified outcome → learning
```

## Planned Tools

- `get_vehicle_state(case_id)` — returns operating mode, behavioral-deviation score, signal confidence, context confidence, persistence, and available telemetry.
- `get_fault_codes(case_id)` — returns available DTCs and diagnostic metadata.
- `get_service_history(vehicle_id)` — returns relevant prior service events and repairs.
- `request_targeted_telemetry(case_id, signals)` — simulates collection of additional evidence when root-cause confidence is insufficient.

## Guardrails

1. Structured vehicle evidence is the source of truth.
2. Behavioral anomaly is not equivalent to a fault diagnosis.
3. Missing evidence must be surfaced explicitly.
4. The agent may recommend evidence collection before diagnosis.
5. Consequential service actions require technician review.
6. The execution trace should preserve which evidence and tool results supported each recommendation.

## Initial Demonstration Scenario

The first scenario will reuse **Parked Energy & Wake Intelligence**:

- Expected operating mode: `QUIESCENT`
- Behavioral deviation: high
- Signal/context confidence: high
- Persistence: high
- Root-cause confidence: low

Expected agent behavior: recognize that the evidence supports concern but not a root-cause diagnosis, call `request_targeted_telemetry()`, reassess the returned evidence, and recommend the next diagnostic action for technician review.

## Planned Implementation

Python + LLM API with tool/function calling. The implementation will be kept intentionally transparent so each tool invocation, returned evidence state, and final recommendation can be inspected.

Potential later extensions include retrieval over service documentation, reusable diagnostic skills, additional vehicle-health domains, and evaluation against verified technician outcomes.
