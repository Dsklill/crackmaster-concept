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

---
