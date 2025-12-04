# 📋 ARCHITECTURE DECISION RECORD (ADR) - RECOMMENDATION

**Date**: 2025-12-05  
**Status**: ⏳ **AWAITING USER CONSENT**  
**Recommended for Documentation**: 3 Significant Architectural Decisions

---

## Decision Summary

During the `/sp.plan` workflow, three architecturally significant decisions were identified that warrant formal ADR documentation. These decisions:

- ✅ Have **long-term system impact** (affect core design for 13+ weeks)
- ✅ Involved **multiple viable alternatives** (considered 2-3 options per decision)
- ✅ Are **cross-cutting** (influence design across perception, planning, control layers)

---

## Decision 1: Distributed ROS 2 vs. Monolithic Design

**Context**: How should the humanoid robot system be architected?

**Options Considered**:
1. **Distributed ROS 2** (chosen) — Multiple independent nodes (perception, planning, control) communicating via pub/sub + services
2. Monolithic Python Application — All logic in single process
3. ROS 1 (legacy) — Older middleware with different design patterns

**Decision**: **Distributed ROS 2**

**Rationale**:
- **Parallelization**: 3 teams can work independently on perception, planning, control nodes
- **Testability**: Each node testable in isolation before integration
- **Sim-to-Real**: Same ROS 2 code runs in Gazebo simulation and on real hardware (Jetson)
- **Reusability**: Nodes can be repurposed for other robots (modularity)
- **Maintainability**: Clear interfaces (topics/services/actions) between components

**Tradeoffs**:
- Complexity: Multiple processes, pub/sub synchronization, distributed debugging
- Latency: Inter-process communication slightly slower than monolithic (but <10ms acceptable)
- Learning curve: Team must understand ROS 2, DDS middleware

**Consequences**:
- ✅ Enables parallel development (Phase 1: perception + planning simultaneous)
- ✅ Allows safe testing (each node independently verified before full integration)
- ⚠️ Requires well-defined ROS 2 contracts (documented in contracts/)
- ⚠️ Debugging distributed system more complex (multi-terminal, rqt_graph tools)

---

## Decision 2: LLM-Based Intent Parsing vs. Rule-Based

**Context**: How should natural language voice commands be converted to actionable intents?

**Options Considered**:
1. **LLM-Based (GPT-4)** (chosen) — Large language model parses user intent, decomposes multi-step tasks
2. Rule-Based FSM — Finite-state machine with hand-coded rules ("if 'pick' then action=grasp")
3. Hybrid — HFSM (Hierarchical FSM) combining rule engine + fallback to LLM

**Decision**: **LLM-Based (GPT-4)**

**Rationale**:
- **Flexibility**: Handles infinite command variations (e.g., "grab that cup", "get the cup", "pick up the red cup on the table")
- **Multi-Step Decomposition**: Naturally understands "pick up cup and place in sink" → [navigate, grasp, navigate, place, verify]
- **Semantic Understanding**: GPT-4 understands context, constraints, spatial relationships
- **Maintenance**: No need to maintain growing rule database (extensible without code changes)

**Tradeoffs**:
- **Cost**: $0.03-0.10 per API call (~12 calls/hour = $0.40-1.20/day for capstone use)
- **Latency**: 2-3s response time (async; acceptable for non-urgent commands)
- **Dependency**: Requires OpenAI API access + internet connection (mitigated by Llama-2 fallback)

**Alternatives Rejected**:
- ❌ Rule-based FSM: Unable to handle "pick up the cup" vs. "grab the cup" without duplicating rules
- ❌ HFSM: Added complexity for marginal benefit; GPT-4 already handles most cases

**Consequences**:
- ✅ Users can speak naturally (no need to learn syntax like "grasp(cup, force=2.5)")
- ✅ Easy to add new commands (just speak; no code changes)
- ⚠️ Cost control required (API quota limits in .env)
- ⚠️ Fallback strategy needed (Llama-2 local model if API fails)

---

## Decision 3: Simulation-First Verification vs. Hardware-First

**Context**: How should the capstone system be developed and tested?

**Options Considered**:
1. **Simulation-First** (chosen) — Gazebo 11 primary; real hardware optional post-capstone
2. Hardware-First — Test on real robot from Week 1
3. Parallel Simulation + Hardware — Both simultaneously (resource-intensive)

**Decision**: **Simulation-First**

**Rationale**:
- **Safety**: Mistakes in Gazebo don't break expensive hardware (Jetson, actuators)
- **Velocity**: Iterate rapidly without waiting for hardware setup/deployment
- **Reproducibility**: Simulation is deterministic (same input → same output every time)
- **Scalability**: Multiple teams can work on same codebase without queuing for hardware
- **Debugging**: Easy to inspect internal robot state, replay scenarios, step through simulation

**Tradeoffs**:
- **Reality Gap**: Simulation may diverge from real hardware (friction, noise, latency)
- **Real Feedback**: Delayed hardware validation until Phase 3
- **Domain Randomization**: Extra effort required to train models for sim-to-real transfer

**Alternatives Rejected**:
- ❌ Hardware-First: Too risky (potential robot damage, limited hardware access, slow iteration)
- ❌ Parallel Sim+HW: Doubles development cost (would need 2 robot setups + 2 Jetson boards)

**Consequences**:
- ✅ Safe rapid iteration in Gazebo (Phase 0-2)
- ✅ Real hardware testing in Phase 3 (controlled environment)
- ⚠️ Must verify sim-to-real transfer (domain randomization, quantized models)
- ⚠️ Real hardware surprises possible (calibration, wear, environmental factors)

**Mitigation**: Phase 2 includes quantized model testing on Jetson simulator before Phase 3 hardware deployment.

---

## ADR Suggestion Summary

📋 **Architectural decisions detected**:

1. **Distributed ROS 2 vs. Monolithic** — Long-term system modularity, team collaboration
2. **LLM-Based Intent vs. Rule-Based** — Flexibility, maintenance, cost-benefit
3. **Simulation-First vs. Hardware-First** — Safety, development velocity, reality gap

These decisions should be documented formally to:
- Preserve reasoning for future team members
- Facilitate design reviews + audits
- Enable backtracking if assumptions prove incorrect

---

## Recommended Action

**Option A: Create ADRs (Recommended)**
```bash
/sp.adr "Distributed ROS 2 Architecture"
/sp.adr "LLM-Based Intent Parsing"
/sp.adr "Simulation-First Verification"
```

**Option B: Skip ADRs**
- Proceed directly to `/sp.tasks` task generation
- ADRs can be created retrospectively if needed

---

## User Consent Required

**Question**: Would you like to document these architectural decisions using `/sp.adr` commands?

- **Yes**: Run the 3 ADR commands above (5 min each, ~15 min total)
- **No**: Proceed directly to `/sp.tasks` task generation

This is a **non-blocking decision** — either path leads to successful task generation.

---

**Created**: 2025-12-05
