I propose a system for addressing billions of node trees with high r/w throughput
with evenly distributed range sharding with round‑robin merge at query time.

Where : 
- root node id is linearly growing `u64`
- descendant node ids are derived from the root
    - one-to-one : share the same id as their owning node
    - one-to-many : Vec/Option - composites derived from the owning node id and a current node index as `u16` eg. `blockHeight||txIndex||utxoIndex||assetIndex`
- all possible node descendant ids :
    - u64||u16
    - u64||u16||u16
    - u64||u16||u16||u16
- it is used by storage engines to address and shard entities and their descendants

Such that :
- Key has rich interface for encoding/decoding, serialization/deserialization, comparison, hashing, partitioning, sampling, etc.
  - api that returns : depth, index, total_index_including_root, anything that can be derived only from the composite key numbers
- It allows for evenly distributed range sharding with round‑robin merge at query time, for example :
    - all id levels : `u64||u16` `u64||u16||u16` `u64||u16||u16||u16` are distributed evenly into N shards
    - different levels are stored in-memory or on-disk separately by storage engines

Given that :
- rich interface helps storage engine to : 
  - keep it in a hash map
  - compare it for sorting (concatenated LE byte representation should be comparable as is)
  - sample it for testing
  - serialize/deserialize it for on-disk storage
  - encode/decode it for human readability
  - partition it for sharding