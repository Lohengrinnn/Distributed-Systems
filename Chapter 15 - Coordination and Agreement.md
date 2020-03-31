# Chapter 15 Coordination and Agreement

- Distributed mutual exclusion
- Election
- Multicast - reliable, ordering
- Consensus

## Distributed mutual exclusion

We assume that the system is ***asynchronous***, that processes do not fail and that message delivery is ***reliable***

### Operations

enter -> access -> exit

### Requirements
ME1: (safety) no race condition, one process accesses shared resources
ME2: (liveness) no process starve to death
ME3: (ordering) if p<sub>1</sub> req -> p<sub>2</sub> req, p<sub>1</sub> should acquire token before p<sub>2</sub>

### Criteria
1. Bandwidth: proportional to number of messages sent in enter & exit operation

2. Client delay: during enter and exit operations

3. Throughput: it's greater when synchronization delay is shorter.
   (synchronization delay is delay between one process exiting and another process entering)

### Algorithms
- Central server
- Ring-based algorithm
- Multicast and logical clocks
- Maekawa's voting algorithm



## Election

Voting a process with the largest identifier.

1. election: propose election

2. answer: response to an election message

3. Coordinater: announce the identity of the elected process

### Requirements
E1: (safety) **single elected** A process participant has elected<sub>i</sub> = ⊥ or P, P is with the largest identifier.
E2: (liveness) **each agree to elected** All processes participants are either elected<sub>i</sub> ≠ ⊥ or crash. 

### Bully algorithm

it assumes that message delivery between processes is ***reliable***.

#### priori knowledge

**ring-based algorithm:**	each knows only how to communicate with its neighbor.

**bully algorithm:**				each knows which which processes have higher identifiers, it can communicate with all such processes.

![election_bully_algorithm](https://github.com/Lohengrinnn/Distributed-Systems/blob/master/images/c15_8_bully_algorithm.png?raw=true)

Stage 1: Process *p*1 detects the failure of the coordinator *p*4 and announces an election.
Stage 2: On receiving an *election* message from *p*1 , processes *p*2 and *p*3 send *answer* messages to *p*1 and begin their own elections; *p*3 sends an *answer* message to *p*2 , but *p*3 receives no *answer* message from the failed process *p*4. 
Stage 3: It therefore decides that it is the coordinator. But before it can send out the *coordinator* message, it too fails.
Stage 4: When *p*1 ’s timeout period *T* expires (which we assume occurs before *p*2 ’s timeout expires), it deduces the absence of a *coordinator* message and begins another election. Eventually, *p*2 is elected coordinator.

## Group communication (multicast)

The system under consideration contains a collection of processes, which can communicate ***reliably*** over one-to-one channels.

### Reliable multicast

Integrity:		a correct process p delievers a message m at most once.

Validity:		a correct process multicast(g, m) -> deliver(m).

Agreement:	a correct process deliever(m), all other processes deliever(m).

Validity and Agreement together amount to an overall liveness requirement.

#### reliable multicast over Basic multicast

each processes received a new B-multicast, B-multicast it to the group.

***drawback***: Each message is sent ｜*g*｜ times to each process.

### Ordered multicast

FIFO:		multicast(g, m) before multicast(g, m') ,  deliever(m) before deliever(m')

Casual:	multicast(g, m) -> multicast(g, m') , deliever(m) before deliever(m')

Total:		a process deliever(m) before deliever(m'), all others are the same

## Consensus in general

### Consensus

#### Actions

undecided state -> propose -> exchange -> set decision variable -> decided state

#### Requirements

Termination:	eventually each correct processes sets its decision variable

Agreement:	the decision value of all correct processes is the same

Integrity:		If the correct processes all proposed the same value, then any correct process in the decided state has chosen that value

### * Byzantine generals problem

Differ from consensus, only one commander proposes, other commanders are to agree upon.

#### Requirements

Termination:	as consensus

Agreement:	as consensus

Integrity:		if the commander is correct, all correct processes decided on the value that the commander proposed

### Interactive consistency

**alias decision vector**: every process proposes a single value. all correct processes agree on a vector of values, one for each process.

#### Requirements 

Termination:	as consensus

Agreement:	as consensus

Integrity:		if p<sub>i</sub> is correct, then all correct processes decide on v<sub>i</sub> as ith component of their vector.

## Distributed Transaction

### 2 Phase Commit

​     **Coordinator**                                                      **Paricipant**

1    writes begin_commit in log

2    sends "prepare" message                              2.1 "vote-commit" enters "**READY**" state

​                                                                                  2.2 "vote-abort" 

3.1 Receives "vote-commit", sends "global-commit"

3.2 Receives "vote-abort", sends "global-abort" 

4    enters "**COMMIT**" state                                  4     enters "**COMMIT**" state

