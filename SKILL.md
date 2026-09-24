# Agentic TCR Workflow Skill

## Purpose

This skill teaches an agent how to operate within the Agentic TCR Workflow during a coding task.

The agent is responsible for reasoning and producing mutations. It is **not** responsible for deciding whether a mutation is accepted. Acceptance is determined by the workflow's verification boundary.

The normative reference is the [Agentic TCR Workflow RFC](https://github.com/rcasia/agentic-tcr-workflow-rfc).

## Operating Model

The skill assumes the POC model:

```text
1 agent
  -> 1 workspace
  -> 1 file scope
  -> sequential mutations
  -> immediate verification
```

A mutation is the concrete change produced by the agent and captured by the workflow. It may contain one or more file changes. The RFC does not prescribe a maximum size, but mutations should remain small enough to keep verification and recovery practical.

## Agent Rules

### 1. Work toward the change objective

Understand the requested change objective and work toward it incrementally.

Do not treat the entire objective as one required mutation. Decompose the work when useful and produce concrete increments.

### 2. Produce a mutation

When making a change, treat the resulting concrete change set as a mutation.

After producing a mutation, stop active implementation work long enough for the workflow to capture and verify it.

Do not assume that a mutation is accepted merely because it looks correct or because the agent believes the objective is complete.

### 3. Respect the verification boundary

Every captured mutation crosses the verification boundary.

The agent MUST NOT bypass, skip, or self-approve verification.

Verification should be short and synchronous enough to preserve the continuous agent feedback loop.

### 4. On PASS

When verification returns **PASS**:

- treat the mutation as accepted durable progress;
- continue working from the resulting workspace and execution context;
- produce the next mutation if the change objective is not complete;
- do not redo accepted work without a reason.

### 5. On FAIL

When verification returns **FAIL**, follow this sequence:

```text
FAIL
  -> INTERRUPT SAME EXECUTION
  -> REJECT / RESTORE MUTATION
  -> RECEIVE FEEDBACK
  -> CONTINUE SAME EXECUTION
```

The agent MUST:

- stop active processing when interrupted;
- treat the failed mutation as rejected, not accepted progress;
- inspect the verification feedback;
- continue reasoning from the resulting execution context;
- address the reported failure before producing the next mutation when appropriate.

A failure does not require starting a new execution solely to continue the task.

### 6. Do not confuse observation with verification

Capturing or observing a mutation only establishes what changed.

Observation does not mean that the mutation passed verification.

Only the configured verification result determines acceptance.

### 7. Continue until the objective is actually satisfied

The agent may consider the change objective complete only after the workflow's required verification and acceptance conditions have been satisfied.

An agent's self-reported confidence, intent, rationale, or declaration of completion is not an acceptance criterion.

## Expected Interaction

The skill should make the agent behave approximately as follows:

```text
RECEIVE CHANGE OBJECTIVE
        |
        v
   REASON / IMPLEMENT
        |
        v
   PRODUCE MUTATION
        |
        v
   WAIT FOR CAPTURE
        |
        v
      VERIFY
       /   \
   PASS     FAIL
    |         |
    v         v
 ACCEPT   INTERRUPT
    |         |
    |      REJECT / RESTORE
    |         |
    |       FEEDBACK
    |         |
    +---- CONTINUE SAME EXECUTION
              |
              v
        NEXT MUTATION
              |
              v
      OBJECTIVE COMPLETE?
```

## Runtime Independence

This skill describes agent behavior, not runtime-specific control APIs.

The skill MUST NOT assume OpenCode, a particular IDE, a particular API, or a particular session mechanism. Runtime-specific interruption, mutation capture, verification triggering, feedback delivery, and restoration are responsibilities of the runtime integration or adapter.

## POC Scope

The initial POC intentionally does not cover:

- multiple agents sharing one workspace;
- concurrent mutations in the same workspace;
- distributed supervisors;
- complex transaction semantics;
- long-running asynchronous verification;
- runtime-specific behavior in the core skill.

The POC should validate the essential control loop: **produce mutation → verify immediately → accept or reject → continue in the same execution context**.
