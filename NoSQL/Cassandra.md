# Cassandra 

## Hash Function
- md5
- murmur3 hash
- order hash

## Gossip Protocol 
- Membership 
- Failure Detection 

- Fan Number 
- Time To Live (TTL)

## Problems 
Cassandra is a AP system according to the CAP theorem 

最终一致性

Tunable Consistency 
- N is the number of replicas 
- W is the consistency level of write operations（接收到W个写成功就认为是consistent -> 牺牲consistency满足availability）
- R is the consistency level of read operations 

W + R > N -> strong consistency

Quorum = N / 2 + 1 = 3


- Read Repair 
- Hinted Handoff
- Anti-Entropy Repair 
  - To routinely maintain node health 
  - To recover a node after a failure while bringing it back into the cluster 
  - To update data on a node containing data 
  
How to detect consistency? -> Merkle Trees 

### Load Balance
Solution:
- Virtual Nodes 
- Moves Nodes 

