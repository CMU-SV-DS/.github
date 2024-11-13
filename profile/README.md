# Hi there 👋


## Project Structure

Raft/ 2PC

- fully functional raft is under sadds-raft main branch https://github.com/CMU-SV-DS/sadds-raft/tree/main
  - instructions: https://github.com/CMU-SV-DS/sadds-raft/blob/main/README.md
- fully functional 2PC is under sadds-raft refactor-structure-kejie branch (not mege into main yet) https://github.com/CMU-SV-DS/sadds-raft/tree/refactor-structure-kejie
  - Instructions: https://github.com/CMU-SV-DS/sadds-raft/blob/refactor-structure-kejie/StartInstagram.md

Data shardning
- fully functional sadds-service (backend of instagram story) [proxy-dev branch](https://github.com/CMU-SV-DS/sadds-service)
  - go into sadds-service-proxy and follow the instruction [here](https://github.com/CMU-SV-DS/sadds-service/tree/proxy-dev/sadds-service-proxy)
  - run sadds-service https://github.com/CMU-SV-DS/sadds-service/blob/proxy-dev/README.md

 - Raft dashboard
   - raft-client switch to sharding-updates branch https://github.com/CMU-SV-DS/sadds-raft-client/tree/sharding-updates
   - Raft-monitor

Instagram client (for user)

- Backend is under sadds-service main branch https://github.com/CMU-SV-DS/sadds-service
- frontend is under sadds-client main branch https://github.com/CMU-SV-DS/sadds-client

- instructions: should run with raft/2pc, follow instructions here: https://github.com/CMU-SV-DS/sadds-raft/blob/main/README.md



## Raft / 2PC

### Raft Implementation

- Leader Election:
  - Election Timeout: If a follower node doesn’t receive communication (a "heartbeat") from a leader within a set time, it assumes the leader has failed.
  - The follower becomes a candidate and requests votes from other nodes to become the new leader.
  - Voting: Each node can vote for only one candidate per term. If a candidate receives a majority of votes, it becomes the new leader.
  - The leader then sends regular heartbeats to keep followers in sync and prevent new elections.
- Log Replication:
  - Leader Receives Client Requests: The leader accepts client requests, adds entries to its log, and sends these entries to followers.
  - Replication to Followers: Followers append the entries to their logs and acknowledge receipt.
  - Commit: Once a majority of followers have appended an entry, the leader marks it as committed and applies it to its state machine.
  - The leader then informs followers of committed entries so they apply these changes to their own state machines, ensuring consistency.
- Safety and Consistency:
  - Commit Guarantee: Raft ensures that committed entries are not lost as long as a majority of nodes are alive.
  - Consistency: All nodes eventually apply the same sequence of log entries in the same order, so they maintain a consistent state across the distributed system.

### 2PC Implementation

- Prepare Phase:
  - The coordinator sends a "prepare" request to each participant, asking if it can commit the transaction.
  - Each participant checks if it can complete the transaction (e.g., has enough resources, no conflicts) and responds with a "yes" (prepared to commit) or "no" (cannot commit).
  - If any participant sends a "no," the coordinator decides to abort the transaction.
- Commit Phase:
  - If all participants responded "yes," the coordinator sends a "commit" request, and each participant commits the transaction.
  - If any participant responded "no" in the prepare phase, or if a failure occurred, the coordinator sends a "rollback" request, and each participant rolls back its part of the transaction.
  - Each participant acknowledges the commit or rollback to the coordinator, ensuring all nodes reach the same final state.


## Data Shardning

### Implementation logic

- Data Split into Two Tables:
  - Follow Table: Stores entries where one user follows another. This table is sharded based on the name (or unique identifier) of the follower (user who is following someone else).
  - Following Table: Stores entries for users who are followed by others. This table is sharded based on the name (or unique identifier) of the followee (user who is being followed).
- Data Insertion:
  - When a user (User1) follows another user (User2), two entries are created:
    - An entry in the Follow Table indicating that User1 follows User2. This entry is stored in the shard assigned to User1.
    - An entry in the Following Table showing that User2 is followed by User1. This entry is stored in the shard assigned to User2.
- Data Retrieval:
  - When viewing a user’s profile, only one shard needs to be accessed to retrieve both the user's following and followers data:
    - If you’re looking up the following list (people the user follows), you only query the shard based on the user’s name in the Follow Table.
    - If you’re looking up the follower list (people following the user), you only query the shard based on the user’s name in the Following Table.

 ### Benefits of the design

- Efficient Retrieval: By storing "follow" and "follower" data separately and sharding based on usernames, you can retrieve a user’s entire following or follower list from a single shard, reducing the need for complex cross-shard queries.
- Scalability: Sharding by usernames distributes data evenly across shards as the number of users grows, allowing for horizontal scalability and avoiding concentration of data in any single shard.
- Balanced Write Load: Each follow action is written to two distinct shards, spreading the write load across the database system and preventing overload on any single shard.



