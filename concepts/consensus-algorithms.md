# 🤝 Consensus Algorithms: Paxos & Raft

In a distributed system, how do multiple independent nodes agree on a single value or a sequence of events, especially when nodes fail or the network is unreliable? This is the **Consensus Problem**. Algorithms like **Paxos** and **Raft** provide the mathematical foundation for building strongly consistent, fault-tolerant systems.

> **Staff SRE Insight:** "Consensus is the 'engine' of distributed consistency. Without it, you cannot have a leader, you cannot have distributed locks, and you cannot have a reliable service discovery system like etcd or Zookeeper. It’s expensive in terms of latency, but it’s the only way to achieve a 'Single Source of Truth' in a chaotic network."

---

## 🏗️ 1. The Core Objective
The goal of a consensus algorithm is to ensure:
- **Validity:** If a value is decided, it must have been proposed by a node.
- **Agreement:** All non-faulty nodes eventually decide on the same value.
- **Termination:** Every non-faulty node eventually reaches a decision.
- **Fault Tolerance:** The system remains operational as long as a majority (Quorum) of nodes are healthy ($N/2 + 1$).

---

## 🚀 2. Raft: Understandable Consensus
Raft was designed as a more practical and understandable alternative to Paxos. It decomposes the consensus problem into three sub-problems:

### A. Leader Election
Nodes can be in one of three states: **Follower**, **Candidate**, or **Leader**. If a Follower stops hearing from a Leader, it starts an election.


### B. Log Replication
The Leader accepts client requests, appends them to its log, and then replicates those logs to the Followers. Once a majority of Followers acknowledge the log entry, it is considered **Committed**.

### C. Safety
Raft ensures that if any node has applied a particular log entry to its state machine, no other node can apply a different command for that same log index.

---

## 🛠️ Paxos: The Foundation
Paxos is the older, more complex predecessor to Raft. It is widely used in systems like **Google Spanner** and **Cassandra (Lightweight Transactions)**.
- **Roles:** Proposers, Acceptors, and Learners.
- **Phases:** Prepare/Promise and Accept/Accepted.
- **Trade-off:** Paxos is mathematically robust but notoriously difficult to implement correctly in production without edge-case bugs.

---

## ⚖️ Architectural Trade-offs

| Feature | Consensus (Raft/Paxos) | Gossip Protocols |
| :--- | :--- | :--- |
| **Consistency** | Strong (Linearizable) | Eventual |
| **Performance** | Lower (Requires Quorum RTT) | Higher (Asynchronous) |
| **Scalability** | Limited (Usually 3, 5, or 7 nodes) | Massive (Thousands of nodes) |
| **Use Case** | Leadership, Metadata, Locks | Service Discovery, Health Checks |



---

## 🛠️ Real-World Application
- **etcd / Zookeeper:** Use consensus to store critical configuration and perform leader election for Kubernetes or Kafka.
- **Distributed Databases:** Use Paxos/Raft to ensure that all replicas of a specific data shard agree on the order of writes.

---

## 📊 SRE Performance Metrics

- **Election Duration:** How long the cluster is "leaderless" during a failure. This is the time where writes are blocked.
- **Commit Latency:** The p99 time it takes for a proposal to be acknowledged by a quorum of nodes.
- **Heartbeat Jitter:** Monitoring the stability of the leader's heartbeat to prevent "unnecessary" re-elections (flapping).
- **Quorum Health:** Percentage of nodes currently participating in consensus. If you lose more than $(N-1)/2$ nodes, the system halts to protect data integrity.

---
