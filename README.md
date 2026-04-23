# crackmaster-concept
An immature concept of AI-assisted vulnerability analysis architecture. No code, only ideas.

## Preface

This is just an immature personal idea with no real results. Writing it down is just to organize your thoughts, and it may be of some reference value to people who do similar things. 

## Initial Thoughts

The initial idea was naïve: to make a "ghost virtual machine" based on the JVM, full instruction transparent hooks, independent memory management, and completely independent host control. The middle layer is full instruction capture, and the hub is an AI inference module called CrackMaster, which can automatically output utilization ideas. The execution layer uses MCP as a skill pipeline, plus closed-loop self-calibration. The goal is to allow AI to mine vulnerabilities and automatically generate verification code. 

Later, it was soon discovered that the JVM could not do instruction-level isolation at all, the performance loss of full instruction hooks was unacceptably large, and the AI was unstable to generate even simple exploit chains, let alone real-time inference. 

So I started doing subtraction. 

## Framework made by subtraction

Ditch the JVM in favor of QEMU/KVM hardware virtualization. 

Abandon full instruction hooks and only collect key API and memory operations. 

Ditch AI real-time decision-making and instead fix a five-tier pipeline. 

Ditch the AI-controlled flow and let it assist asynchronously at specific nodes. 

Abandon the illusion of a "digital twin" and only generate verifiable vulnerable point hypotheses. 

The final framework looks like this: 

**Information collection layer**: Static analysis (architecture, import and export, strings, dependency libraries), dynamic ingestion (API call sequences, heap operations, exceptions), environmental detection (ASLR, CFG, etc.). The AI is only asked: "Which APIs do you think should be monitored based on static information?" Output 'raw_profile.json'. 

**Knowledge Building Layer**: Give the collected information to the large model and output vulnerable hypotheses (type, confidence, evidence, target details). Do not pursue absolute correctness, only output verifiable conjectures. Output 'hypotheses.json'. 

**Test Inference Layer**: For each hypothesis, the AI selects skills, orchestrates order, populates parameters from an atomic skill library ('snapshot_restore', 'hook_function', 'modify_memory', 'call_api', etc.). Without using a generic template, each test sequence is tailored to that goal. Output 'test_plan.json'. 

Interaction validation layer: This layer has no AI. Create a separate QEMU virtual machine for each test case (a copy-on-write clone based on a clean snapshot), mechanically execute the skill sequence, record the results, and destroy the environment. Output 'verification_results.json'. 

**Closed-loop iteration layer**: The AI analyzes the test results and updates the false setting reliability to decide whether to continue testing, correct knowledge, or end. Output updated hypotheses and reports. 

The master control is just a deterministic script that calls five layers in sequence, passes JSON files, handles timeouts and breakpoints. Not smart, even a bit dull. 

That's all there is to it. It breaks down the complex problem of vulnerability analysis into five links that can be debugged independently, and the AI is limited to specific nodes, and deterministic code takes over when it leaves the cage. 

- **Acknowledge the limitations of AI**: AI hallucinates, times out, and diverges. Instead of trying to "fix" AI, it cages it with architecture. 
- **Breaking Down Complexity**: Instead of requiring AI to solve everything in one step, each layer does only one thing, with clear inputs and outputs for everything. 
- **Pursue reliability, not speed**: Independent virtual machine for each test case, persistence, breakpoint continuation, retry and downgrade. Rather slow than stable. 

## This is an idea that has not yet been realized

So far, there has not been a single line of code, and there has not been a single successful verification use case. If one day someone really follows this idea, they may step on many pitfalls and find that it doesn't work at all; but it may also succeed. 

Writing it down is just to sort out your thoughts, and it may be a little helpful for you who are thinking about similar questions. 


flowchart TD
    subgraph InformationCollection ["Information Collection Layer (Perception)"]
        A1[Static Analysis Engine<br/>- Binary Parsing<br/>- Decompilation<br/>- String Extraction<br/>- Import Table]
        A2[Dynamic Tracing Engine<br/>- API Hook<br/>- Component Interaction<br/>- Network Communication<br/>- File Operations]
        A3[Environment Detection Engine<br/>- Dependency Analysis<br/>- Config Parsing<br/>- Language Detection<br/>- Framework Fingerprint]
    end

    B["Knowledge Construction Layer (Reasoning)<br/>AI Analysis Engine (Local LLM + Knowledge Graph)<br/>- Identify Software Architecture<br/>- Extract Component List & Functions<br/>- Infer Business Logic<br/>- Generate Tech Stack Summary<br/>Output: Structured Knowledge Base"]
    
    C["Test Reasoning Layer (Decision)<br/>Skill Scheduler + MCP Code Assistant<br/>- Generate Test Hypotheses<br/>- Orchestrate Test Skills<br/>- Generate Test Code for MCP Execution"]
    
    D["Interactive Verification Layer (Execution)<br/>Target Environment (VM/Container/Process)<br/>- Receive Test Commands (MCP)<br/>- Execute Test Code<br/>- Feedback Results"]
    
    E["Closed-Loop Iteration (Learning)<br/>- Success: Strengthen Knowledge Hypotheses<br/>- Failure: Correct & Generate New Tests<br/>- Unknown Behavior: Trigger Deep Collection"]

    InformationCollection -->|Raw Data: API Sequences, Calls, Code Snippets| B
    B -->|Components, API Dependencies, Logic Hypotheses| C
    C -->|Test Scripts, Test Parameters| D
    D -->|Verification Results & New Data| E
    E --> B










┌─────────────────────────────────────────────────────────────────┐
│ Orchestrator – Deterministic Sequential Scheduling               │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ Information Collection Layer                                      │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────────┐  │
│ │Static Analysis│ │Dynamic Capture│ │Environment Detection│ │AI Focus API Suggestions│
│ └──────────┘ └──────────┘ └──────────┘ └────────────────────┘  │
│                       Output: raw_profile.json                   │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ Knowledge Construction Layer                                     │
│ ┌────────────────────────────────────────────────────────────┐  │
│ │ LLM Analysis → Generate Vulnerability Hypotheses (Type, Confidence, Evidence, Target Details) │
│ └────────────────────────────────────────────────────────────┘  │
│                       Output: hypotheses.json                   │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ Testing & Reasoning Layer                                        │
│ ┌────────────────────────────────────────────────────────────┐  │
│ │ AI Select Atomic Exploits + Arrange Execution Order + Fill Parameters (URL/API/Conditions) │
│ └────────────────────────────────────────────────────────────┘  │
│                       Output: test_plan.json                     │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ Verification Layer (AI-Free Execution)                           │
│ ┌────────────────────────────────────────────────────────────┐  │
│ │ For each test case: Spawn QEMU Sandbox → Execute Attack Chain → Record Crash Log → Destroy Instance │
│ └────────────────────────────────────────────────────────────┘  │
│                       Output: verification_results.json          │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ Closed-Loop Iteration Layer                                      │
│ ┌────────────────────────────────────────────────────────────┐  │
│ │ AI Result Analysis → Update Confidence Score → Retest / Backtrack / Terminate Task │
│ └────────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
                         ┌─────────────┐
                         │ Final Report│
                         └─────────────┘





---
