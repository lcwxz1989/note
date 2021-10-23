# _

## 1. Terminology

### 1.1. Term

Terms act as a logic clock in Raft

1. Election
2. AppendEntries

### 1.2. Roles

leader
candidate
follower

### 1.3. State

#### 1.3.1. Persistent All Server

1. currentTerm: latest term server has seen (initialized to 0 on first boot, increases monotonically),basis for every action, becase every action must in same term;
2. votedFor: candidateId that received vote in current term (or null if none);
3. log[]: log entries; each entry contains command for state machine, and term when entry was received by leader (first index is 1).

**Tips**
1. `currentTerm` and `votedFor` is context on which the cluster must base;
2. log structure: index, term, command.

#### 1.3.2. Volatile All Server

1. commitIndex: commitIndex means have recieved by majority of machine;
2. lastApplied: have processed by state machine.

**Tips**

1. used by the server itself.

#### 1.3.3. Volatile Leader

nextIndex[]: the index will be send to for every followers. Most of the time, `nextIndex` equals `matchIndex` + 1, but it is not always the case. For example, when a leader is initiated, matchIndex is initiated to the 0, while nextIndex is initiated to the last log index + 1;
matchIndex[]: have replicated on majority of machine but not commited. This will happened by append entries in batch, if majority of machine's matchIndex is N, and N is bigger than leader's committedIndex, leader can update committedIndex to N.

**Tips**
1. The core of the leader's job is two, one is to colculate the next `commitIndex` of the cluster and another is to construct append entries;
2. The difference here is because these two fields are used for different purposes: `matchIndex` is an accurate value indicating the index up to which all the log entries in leader and follower match. However, `nextIndex` is only an optimistic "guess" indicating which index the leader should try for the next AppendEntries operation, it can be a good guess (i.e. it equals matchIndex + 1) in which case the AppendEntries operation will succeed, but it can also be a bad guess (e.g. in the case when a leader was just initiated) in which case the AppendEntries will fail so that the leader will decrement nextIndex and retry.

## 2. WorkFlow

### 2.1. RequestVote RPC

#### 2.1.1. Arguments & Results

**Arguments:**
1. term: candidate’s term
2. candidateId: candidate requesting vote
3. lastLogIndex: index of candidate’s last log entry (§5.4)
4. lastLogTerm: term of candidate’s last log entry (§5.4)

**Results:**
1. term: currentTerm, for candidate to update itself
2. voteGranted: true means candidate received vote

#### 2.2.2. Receiver implementation

1. Reply false if `term` < `currentTerm` (§5.1)
2. If `votedFor` is null or candidateId, and candidate’s log is at least as up-to-date as receiver’s log, grant vote (§5.2, §5.4)

#### 2.2.3. Thinking

1. Raft uses the voting process to select a candidate which contains all committed logs of the previous leader's, so the election just like to select a replica of the previous leader.

    **Restriction**
    1. `committed` log means replicated by majority of machines(M1 > N/2)
    2. new leader will voted by majority of machines(M2 > N/2)
    3. the new leader will be retain biggest `latestIndex` in all of the candidate

    **Conclusion**: 
    1. based on *restriction 1* and *restriction 2*, there will be at least one `m machine` both in M1 and M2, of course maybe two and so on, and the machine retain the logs as a redundant of the previous leader's
    2. based on *restriction 3* and *conclusion 1*, the one which is voted will be retain the log as a redundant of the previous leader's
    3. base on *conclusion 2*, there will be just one direction from new leader to followers in log migration
    4. in one word, raft guarantees there is at least one replicate logs in followers as latest as leader's, and this one will be the future leader

### 2.2. AppendEntries RPC

#### 2.2.1. Arguments & Results

**Arguments:**
1. term: leader’s term
2. leaderId: so follower can redirect clients
3. prevLogIndex: index of log entry immediately preceding new ones
4. prevLogTerm: term of prevLogIndex entry
5. entries[]: log entries to store (empty for heartbeat; may send more than one for efficiency) leader’s commitIndex
6. leaderCommit: leader’s commitIndex, leader update it's commitIndex by "If there exists an N such that N > commitIndex, a majority of matchIndex[i] ≥ N, and log[N].term == currentTerm: set commitIndex = N".

**Tips**:
1. `term` and `leaderId` is the basic context;
2. `prevLogIndex` and `prevLogTerm` make true preEntries is the same(like a list structure, every node depend on the preNode), in another words to make sure every state change base on the same prestate;
3. `entries[]` nothing;
4. `leaderCommit` update commitIndex

**Results:**
1. term: currentTerm, for leader to update itself
2. success: true if follower contained entry matching `prevLogIndex` and `prevLogTerm`

#### 2.2.2. Receiver implementation

1. Reply false if `term` < `currentTerm` (§5.1)
2. Reply false if log doesn’t contain an entry at `prevLogIndex` whose term matches `prevLogTerm` (§5.3)
3. If an existing entry conflicts with a new one (same index but different terms), delete the existing entry and all that follow it (§5.3)
4. Append any new entries not already in the log
5. If `leaderCommit` > `commitIndex`, set `commitIndex` = min(`leaderCommit`, index of last new entry)

#### 2.2.3. Thinking

1. In one word, base on the **Tips 1** context to update **Tips 3** and **Tips 4** after make sure **Tips 2**
2. Raft append entries like list data structure to make sure `Log Matching`

    **Restriction**
    1. `committed` means log have been replicated on majority of machine
    2. `applied` means log which is `committed` have be processed by `state machine`
    3. leader will only append log, never change or delete
    4. follower will check `prevLogIndex` and `prevLogTerm` in one request to make sure previous logs in the same
    5. leader handles inconsistencies by forcing the followers' logs to duplicate its own, so conflicting entries in follower logs will be overwritten with entries from the leader's
    6. only log entries from the leader's current term are commited by counting replicas

## 3. References

1. [nextIndex vs matchIndex](https://stackoverflow.com/questions/46376293/what-is-lastapplied-and-matchindex-in-raft-protocol-for-volatile-state-in-server?answertab=votes#tab-top);
