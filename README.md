# Raft Consensus in C++

**Karthik Venugopal · Distributed Systems · USC**

Implemented the Raft consensus lab in C++20 using gRPC and Protocol Buffers: leader election, replicated logs, majority-based commits, and synchronous proposals. The implementation coordinates independent server processes and handles leader changes, disconnected peers, and conflicting log histories.

This repository is a documentation-only portfolio of my individual lab implementation. Source code, course infrastructure, and the subsequent final-project visualizer are not included.

## Implementation

| Component | Behavior |
| --- | --- |
| Leader election | Follower, candidate, and leader roles; randomized election timeouts; RequestVote RPCs; majority voting; and term-based transitions. |
| Heartbeats | Periodic AppendEntries RPCs maintain leadership and reset follower election timers. |
| Log replication | Leaders track each follower's next and matched log indices and replicate commands through AppendEntries. |
| Conflict repair | Followers validate the preceding log entry, truncate conflicting suffixes, and return conflict-term/index hints to accelerate catch-up. |
| Commit advancement | Leaders commit current-term entries after replication to a majority; followers advance from the leader's commit index. |
| Ordered application | A background worker delivers committed commands to the application through a message queue. |
| Proposal APIs | Asynchronous proposals return an index, term, and leadership status. A synchronous proposal API waits on application progress or a leadership/term change. |

## Architecture

Each node hosts a gRPC service and maintains client stubs for its peers. Protocol Buffers define the RequestVote and AppendEntries messages, including log entries and conflict hints.

Election, heartbeat, and application workers run concurrently. A mutex protects consensus state, while condition variables coordinate replication and application progress. Outgoing RPCs execute outside the state lock and use deadlines to bound individual network calls.

```text
Client proposal
      |
      v
Leader appends command
      |
      +---- AppendEntries ----> Follower
      +---- AppendEntries ----> Follower
      |
Majority acknowledges replication
      |
      v
Commit index advances
      |
      v
Committed commands enter the application queue
```

## Failure scenarios and testing

The accompanying lab test suite includes scenarios for:

- Initial elections, reelections, repeated elections, and rapid leader changes.
- Leaders stepping down after observing a higher term.
- Basic agreement and concurrent proposals.
- Agreement with a disconnected follower and failure to agree without a majority.
- Rejoining nodes, conflicting logs, and agreement across network partitions.
- RPC counts and transferred-byte checks.

These describe the tests present in the source project, not a new verified test run or a benchmark result.

## Scope

The source identifies the consensus implementation as Labs 1–3 and specifically labels the synchronous proposal API as Lab 3. This portfolio describes those implemented capabilities rather than asserting completion against an unavailable assignment rubric. The earlier Lab 0 RPC exercises are outside this Raft summary.

Consensus state and log entries are held in memory. Disk persistence, crash-restart recovery, snapshots, log compaction, and dynamic cluster membership are outside this implementation's scope.

## Technologies

C++20, gRPC, Protocol Buffers, CMake, GoogleTest, Abseil, spdlog, multithreading, mutexes, and condition variables.

## Resume description

> Implemented Raft consensus in C++20 with gRPC, including randomized leader election, majority-based log replication, conflict repair, and synchronous proposal handling; worked with a distributed test harness covering leader changes, network partitions, concurrent proposals, and follower rejoin scenarios.
