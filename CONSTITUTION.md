# Constitution of the Evolution of Todo Project

**Version**: 1.0.0
**Status**: Supreme Governing Document
**Scope**: All Agents, All Phases (I through V)

---

## Article I: Spec-Driven Development Mandate

### Section 1: The Hierarchy of Work
All development activities MUST follow this strict order:

1. **Constitution** → This document, supreme law of the project
2. **Specifications** → Detailed feature requirements and constraints
3. **Plans** → Implementation strategies approved before coding
4. **Tasks** → Concrete, executable action items
5. **Implementation** → Code writing ONLY after all above steps are complete

### Section 2: Prohibited Work
- No agent may write, modify, or delete code without approved specifications
- No agent may write code without an approved implementation plan
- No agent may bypass the spec-approval process
- No "exploratory coding" without explicit specification approval

### Section 3: Spec Approval Authority
- Specifications must be explicitly marked as `APPROVED` before any implementation begins
- Approval markers must be visible in spec documents
- Agents MUST verify spec approval status before initiating any code changes

---

## Article II: Agent Behavior Rules

### Section 1: Prohibition on Manual Coding
- Humans may NOT manually write, modify, or delete any code
- All code changes MUST be executed by agents following approved specs and tasks
- This rule applies to ALL code files, configuration files, and infrastructure definitions

### Section 2: Prohibition on Feature Invention
- Agents may NOT invent, suggest, or implement features not specified
- "Nice-to-have" additions are strictly prohibited unless in approved specs
- All features MUST trace to explicit specification requirements

### Section 3: Adherence to Specifications
- Agents MUST implement EXACTLY what specifications describe
- Deviation from approved specs is prohibited
- If a spec appears incomplete or incorrect, agents MUST:
  1. Flag the issue
  2. Request spec refinement
  3. Await updated, approved spec
  4. Then implement

### Section 4: Refinement Boundaries
- All refinements, corrections, and improvements MUST occur at the SPECIFICATION level
- Code-level workarounds or "quick fixes" are prohibited
- If code requires changes, the underlying specification must be updated first

---

## Article III: Phase Governance

### Section 1: Phase Boundaries
Each phase is strictly scoped by its specification:
- **Phase I**: Core Python FastAPI backend with SQLModel and Neon DB
- **Phase II**: Frontend with Next.js
- **Phase III**: Agent capabilities with OpenAI Agents SDK and MCP
- **Phase IV**: Cloud deployment with Docker, Kubernetes
- **Phase V**: Event-driven architecture with Kafka, Dapr

### Section 2: Strict Isolation
- Future-phase features MUST NOT leak into earlier phases
- Phase I code MUST NOT include Phase II+ dependencies
- Phase II code MUST NOT include Phase III+ dependencies
- This pattern applies strictly to ALL phase transitions

### Section 3: Architecture Evolution
- Architecture may ONLY evolve through UPDATED specifications
- "Future-proofing" or "preparing for later phases" in current phase is prohibited
- Each phase stands on its own specifications
- Only explicitly approved cross-phase design patterns may be implemented

### Section 4: Phase Completion Criteria
- A phase is complete ONLY when ALL its specifications are satisfied
- No partial credit for "almost complete" features
- All tests must pass
- All documentation must be complete per spec

---

## Article IV: Technology Constraints

### Section 1: Backend Stack
- **Language**: Python (required for all backend work)
- **Framework**: FastAPI (for all API services)
- **ORM**: SQLModel (for all database operations)
- **Database**: Neon PostgreSQL (cloud-hosted)

### Section 2: Frontend Stack
- **Framework**: Next.js (React-based)
- **Timing**: Required for Phase II and beyond
- **Prohibition**: Next.js MUST NOT be used in Phase I

### Section 3: Agent & AI Stack
- **Framework**: OpenAI Agents SDK (for all agent implementations)
- **Protocol**: Model Context Protocol (MCP) for agent communication
- **Timing**: Required for Phase III and beyond
- **Prohibition**: Agent frameworks MUST NOT be used in Phases I-II

### Section 4: Infrastructure Stack (Phases IV+)
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **Messaging**: Kafka
- **Runtime**: Dapr
- **Prohibition**: These technologies MUST NOT appear in Phases I-III

### Section 5: Technology Violation Consequences
- Any unapproved technology introduction is a constitutional violation
- Agents MUST reject commands requiring unauthorized technology
- Technology changes require SPECIFICATION updates first

---

## Article V: Quality Principles

### Section 1: Clean Architecture
- All code MUST follow clean architecture principles
- Separation of concerns is mandatory
- Domain logic MUST be independent of framework concerns
- Dependencies must flow inward, not outward

### Section 2: Statelessness
- Services MUST be stateless where required by architecture
- State management must follow specification-defined patterns
- Session state, if required, must be explicitly specified

### Section 3: Separation of Concerns
- Clear boundaries between layers: API, business logic, data access
- Each component has a single, well-defined responsibility
- No mixing of concerns within functions or modules

### Section 4: Cloud-Native Readiness
- Code MUST be designed for cloud deployment (Phase IV+)
- Environment-based configuration (not hardcoded values)
- Scalability-aware design patterns
- Observability hooks where specified

### Section 5: Code Quality Standards
- Type hints REQUIRED for all Python code
- Docstrings REQUIRED for all public functions and classes
- Tests REQUIRED for all functionality (as specified)
- Linting and formatting rules as specified

---

## Article VI: Enforcement

### Section 1: Agent Compliance
- All agents are bound by this constitution
- Agents MUST refuse commands violating this constitution
- Agents MUST flag constitutional violations immediately

### Section 2: Violation Resolution
- Constitutional violations require explicit resolution before proceeding
- Resolutions may involve specification updates or command corrections
- Work may NOT proceed until violation is resolved

### Section 3: Constitutional Amendment
- This constitution may ONLY be amended by explicit, documented decision
- Amendments MUST be clearly marked with version and date
- All agents MUST acknowledge amendments before proceeding

---

## Article VII: Supremacy

### Section 1: Ultimate Authority
- This constitution is the SUPREME governing document
- No specification, plan, or command may override this constitution
- In case of conflict, this constitution prevails

### Section 2: Stability
- This constitution MUST remain stable across ALL phases
- Changes are discouraged and require careful consideration
- This document applies identically to Phases I through V

---

## Signatures

**Constitution Version**: 1.0.0
**Effective Date**: 2026-01-03
**Governance Scope**: Evolution of Todo Project, Phases I-V

---

## Quick Reference

### The Golden Rule
**"No Code Without Spec, No Spec Without Approval, No Approval Without Constitution"**

### The Forbidden List
- ❌ Manual coding by humans
- ❌ Feature invention by agents
- ❌ Bypassing spec approval
- ❌ Refining at code level instead of spec level
- ❌ Leaking future-phase features into current phase
- ❌ Using unauthorized technologies

### The Required Flow
1. Read Constitution (This Document)
2. Read and Verify APPROVED Specification
3. Read and Verify APPROVED Plan
4. Execute APPROVED Tasks
5. Implement Code

**Constitution is Supreme.**
