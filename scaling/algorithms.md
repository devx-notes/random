# System Design Algorithms

## 1. Bloom Filter

A bloom filter is a probabilistic data structure that is based on hashing. It is used to test whether an element is a member of a set.

- An empty bloom filter is a bit array of $m$ bits, all of which are initially set to $0$. A bit array is an extremely space-efficient data structure that has each position in the array set to either $0$ or $1$.
- A bloom filter is very much like a hash table in that it will use a hash function to map a key to a bucket. However, it will not store that key in that bucket, it will simply mark it as filled. So, many keys might map to same filled bucket, creating false positives.

> [!NOTE]
> False positives is an error in binary classification in which a test result incorrectly indicates the presence of a condition. It is an acceptable form of classification error in many systems because it does not produce functional errors, only slightly increased latency in some specific cases (unnecessary disk I/O, cache-miss, etc.).

### Use Cases

- Avoid expensive database disk reads by quickly checking if a key exists before accessing the disk. 
- Optimizing cache hit rates and reducing latency
- Distributed databases like [Cassandra](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/bloom_filters.html) use bloom filters to optimize data retrieval in distributed environments. 

## 2. Cuckoo Filter

The Cuckoo Filter is a probabilistic data structure that supports fast set membership testing. It is very similar to a bloom filter in that they both are very fast and space efficient. Both the bloom filter and cuckoo filter also report false positives on set membership.

## 3. GeoHash

Geohashing is a geocoding method used to encode geographic coordinates into digits and letters delineating an area on a map.

## References

- [ByteByteGo | EP150: 12 Algorithms for System Design Interviews](https://blog.bytebytego.com/p/ep150-12-algorithms-for-system-design)
- [Brilliant | Bloom Filter](https://brilliant.org/wiki/bloom-filter/)
- [Brilliant | Cuckoo Filter](https://brilliant.org/wiki/cuckoo-filter/)