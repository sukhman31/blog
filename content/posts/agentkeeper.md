+++
date = '2026-08-12T20:02:14+05:30'
draft = false
title = 'AgentKeeper: Bringing Distributed Systems Wisdom to Multi-Agent Coordination'
tags = ['agentkeeper', 'multi-agent', 'distributed-systems']
+++

When you think about running multi-agent systems, there are way too many cases in which coordination between them can fail, or at least not work as seamlessly as they should be. Let us take an example of running a storm of agents which have a shared pool of work, processing documents in this case. If I have 5 agents deployed to process 50 documents in the queue, how do I make sure they know that other agents exist? How do I make them share the work between them and to split the documents evenly or according to a consensus algorithm? This is not a one-off bug, and also not the only instance where a structure like this would fail. In fact, situations like this get linearly worse as you keep on adding agents. Thankfully, this isn't a new issue, and greater engineers than you and I have come up with solutions for problems like this when dealing with distributed systems. We just need to carry them forward to modern day agentic frameworks.

## Three Failure Modes

Before jumping to the solution, I want to discuss the class of problems that I kept in mind while coming up with AgentKeeper. After I explain them to you, the motivation for the project will become clearer. Multi-agent coordination failure can be broadly classified into 3 failure modes -

1. **Agent storm** - As explained in the example in the previous section, this is N agents sharing a task pool with no liveness signal shared between them.
2. **Split-brain planning** - In this case, we have one master agent orchestrating a group of worker agents. Suppose the agent dies, which agent is going to take its place? If coordination fails, then you have multiple agents orchestrating the same tasks which can create quite the pickle.
3. **Capability hallucination** - This is a scenario in which an agent executes a task which it is not equipped to do; not because it 'overstepped', but because there is nothing stopping it as the appropriate guardrails don't exist. For example, an agent registered only to search the web somehow completes a task that requires filesystem. This calls for a lightweight, but effective, entitlements layer to be introduced but we will get to that later on.

> What is surprising is that these problems were also prevalent during the early 2000s in distributed databases and service fleets. Engineers then came out with solutions like ZooKeeper (sound familiar?) and Chubby, which heavily inspire the project we are discussing today.

## Why We Can't Just Copy ZooKeeper

While frameworks like Langchain and AutoGen are really good at solving how a single agent thinks and acts, they really lack the infrastructure to be able to run multi-agent systems. This is where AgentKeeper comes in; it aims to solve the problem of how a fleet of agents behaves collectively, and it's not just another agent framework. That being said, we can't just drag-and-drop the ZooKeeper architecture onto agents because they violate three assumptions that classical distributed systems make about their actors:

1. **Non-determinism** - Because LLMs are intrinsically non-deterministic, there is no guarantee that given the same input to our system, the output would be the same, so what an agent is capable of or likely to do can drift between same calls. This is why we need active enforcement of what an agent is actually allowed to do and we can't just have a registry that lists agents.
2. **Variable latency** - Again, LLM calls can take anywhere from 50ms to 30 seconds, depending on the model load. This directly breaks the naive heartbeat timeouts you see in many distributed systems, and so there is a danger of evicting agents that are just waiting on a slow inference call.
3. **Natural language interfaces** - Classical systems match capabilities by exact string matching, but this would again break for agents because we need semantic matching capabilities to figure out which of them to select to actually get the job done.

## The Six Subsystems

All this was kept in mind while coming up with the architecture for AgentKeeper, which is what we are going to discuss next. The whole project can be broken down in 6 sub-systems, each designed to tackle all the problems that we have discussed so far -

1. **Agent Registry** - This is where agents can register themselves, they each get a unique ID and a heartbeat. You are only declared dead if you miss enough heartbeats, which is how the rest of the systems know to react. The grace period is generous; 3 times the heartbeat to specifically avoid falsely evicting an agent.
2. **Lease Manager** - This was built specifically to deal with agent storms; a task can only be claimed by one agent and this is an atomic operation. The winner for the action election gets a token which proves that they are the legitimate holder, just so that in case the agent crashes, it can't accidentally interfere with whoever picked the task after it.
3. **Event Bus** - The internal publish-subscribe backbone that helps the subsystems and agents "watch" for specific events and react to them accordingly. This is the real-time information channel that informs many of the critical decisions in the system like an agent being registered or dying, acquisition of a lease and new leader election to name a few.
4. **Leader Election** - This is for when an orchestrator dies, the remaining candidates deterministically pick a new one. This happens instantly and kills split-brain planning.
5. **Capability Index** - This is a lightweight semantic layer built into the project which takes in a task description and matches it to an agent that can actually do it, without hardcoding agent IDs into every task.
6. **Entitlements** - This is the authorization layer where every action an agent takes gets evaluated and authorized. Policies are simple ABAC rules that define what an agent can and can't do. This makes sure that there is authorization and accountability for autonomous agents, not just coordinated efficiency.

![AgentKeeper architecture: Agent Registry, Lease Manager, and Leader Election feed into Capability Index, Entitlement Service, and Task Service, all publishing onto a shared Event Bus](/blog/images/agentkeeper-architecture.png "AgentKeeper architecture — the six subsystems and how they interact")

To make adoption and usage easier, I provide you with the AgentKeeper Python SDK, which you can install easily using pip or the package manager of your choice. It takes about ten lines of code of wrapping code to bolt AgentKeeper onto an existing Langchain or AutoGen setup. I won't go into detail about the SDK here, but you can read more about it here: [github.com/sukhman31/agentkeeper](https://github.com/sukhman31/agentkeeper/blob/main/README.md)

## Does It Actually Work?

So does any of this actually work though? I mean it's good that we can pattern match distributed systems onto multi-agent systems in theory, but it's only great if we get the results that we want. I ran some tests on a set of benchmark tests I built to specifically trigger each of these failure modes to figure out if the problems we actually started off with were solved or not-

| Failure mode | Result |
|---|---|
| **Agent storm** | Without coordination, 20 Langchain workers took ~4000ms to finish a shared pool of 200 tasks, while it only took AgentKeeper ~375ms to do the same. |
| **Leader election recovery** | There was a sub-ms handoff for orchestrator agents on shutdown, a real call-back to the Event Bus point from the previous section. |
| **Capability hallucination** | To test this, I added tasks in the benchmark that required tool access that some agents weren't supposed to have and with no enforcement, the agents did those tasks anyway. Once I switched the entitlement layer on, that problem did not exist anymore. |

## Limitations

That said, all testing so far has been synthetic, not live production traffic, which is a limitation. All testing has been done to prove that the mechanisms work but it doesn't tell you how these things hold up against real LLM API latency patterns. This is the next real validation step. Another limitation is that the leader election isn't real distributed consensus yet like what we have for Raft or Paxos and would not work for a system running across multiple machines with real failure scenarios like disaster recovery, network partitions etc. This is why the current system is best suited to single-node deployments.

## Closing Thoughts

It really goes to show the vision that engineers in the past had that 20-year-old solutions and ideas with no concept of 'reasoning' are still working, for agent fleets nonetheless. However, I feel that agents are going to keep getting weirder and more autonomous and maybe these assumptions that we adapted for don't work 6 months from now. Maybe we might have to come up with something even more novel after all.
