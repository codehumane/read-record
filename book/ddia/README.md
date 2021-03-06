# Designing Data-Intensive Application

> 기록 분량이 많아져서 몇 개의 파일로 나눔. (Derived Data는 정리하지 않을 것)

## [I. Foundations of Data Systems](Foundations-of-Data-Systems.md)

### [Data Models ans Query Languages](https://github.com/codehumane/what-i-learned/blob/master/ddia/Foundations-of-Data-Systems.md#data-models-ans-query-languages)

- [Relational Model Versus Document Model](https://github.com/codehumane/what-i-learned/blob/master/ddia/Foundations-of-Data-Systems.md#relational-model-versus-document-model)
- [Query Language for Data](https://github.com/codehumane/what-i-learned/blob/master/ddia/Foundations-of-Data-Systems.md#query-languages-for-data)

### [Storage and Retrieval](https://github.com/codehumane/what-i-learned/blob/master/ddia/Foundations-of-Data-Systems.md#storage-and-retrieval)

- [Data Structures That Power Your Database](https://github.com/codehumane/what-i-learned/blob/master/ddia/Foundations-of-Data-Systems.md#data-structures-that-power-your-database)
- [Transaction Processing or Analytics?](https://github.com/codehumane/what-i-learned/blob/master/ddia/Foundations-of-Data-Systems.md#transaction-processing-or-analytics)
- [Column-Oriented Storage](https://github.com/codehumane/what-i-learned/blob/master/ddia/Foundations-of-Data-Systems.md#column-oriented-storage)

## [II. Distributed Data](Distributed-Data.md)

### [Replication](https://github.com/codehumane/what-i-learned/blob/master/ddia/Distributed-Data.md#replication)

- [Leaders and Followers](https://github.com/codehumane/what-i-learned/blob/master/ddia/Distributed-Data.md#leaders-and-followers)
- [Problems with Replication Lag](https://github.com/codehumane/what-i-learned/blob/master/ddia/Distributed-Data.md#problems-with-replication-lag)
- [Multi-Leader Replication](https://github.com/codehumane/what-i-learned/blob/master/ddia/Distributed-Data.md#multi-leader-replication)
- [Leaderless Replication](https://github.com/codehumane/what-i-learned/blob/master/ddia/Distributed-Data.md#leaderless-replication)

### [Partitioning](./Distributed-Data.md#partitioning)

- [Partitioning and Replication](./Distributed-Data.md#partitioning-and-replication)
- [Partitioning of Key-Value Data](./Distributed-Data.md#partitioning-of-key-value-data)
- [Partitioning and Secondary Indexes](./Distributed-Data.md#partitioning-and-secondary-indexes)
- [Rebalancing Partitions](./Distributed-Data.md#rebalancing-partitions)
- [Request Routing](./Distributed-Data.md#request-routing)

### [Transactions](./Distributed-Data.md#transactions)

- [The Slippery Concept of a Transaction](./Distributed-Data.md#the-slippery-concept-of-a-transaction)
- [Weak Isolation Levels](./Distributed-Data.md#weak-isolation-levels)
- [Summary](./Distributed-Data.md#summary)

### [The Trouble with Distributed Systems](./Distributed-Data.md#the-trouble-with-distributed-systems)

- [Faults and Partial Failures](./Distributed-Data.md#faults-and-partial-failures)
- [Unreliable Networks](./Distributed-Data.md#unreliable-networks)
- [Unreliable Clocks](./Distributed-Data.md#unreliable-clocks)
- [Knowledge, Truth, and Lies](https://github.com/codehumane/what-i-learned/blob/master/book/ddia/Distributed-Data.md#knowledge-truth-and-lies)

### [Consistency and Consensus](./Distributed-Data.md#consistency-and-consensus)

- [Consistency Guarantees](./Distributed-Data.md#consistency-guarantees)
- [Linearizability](./Distributed-Data.md#linearizability)
- [Ordering Guarantees](./Distributed-Data.md#ordering-guarantees)
- [Dddiaistributed Transactions and Consensus](./Distributed-Data.md#distributed-transactions-and-consensus)

