

| Functional Requirement                                                                                                                                                                                                     | Non-Functional Requirement                                                                                      |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| $\text{SET(key, val, ttl)}$<br>: store a value against unique key                                                                                                                                                          | $\text{Low-Latency}$<br>: $P99$(percentil): $[1, 10]ms$                                                         |
| $\text{GET(key)}$<br>: retrieve a value by key                                                                                                                                                                             | $\text{Scalability}$<br>: Horizontal scaling                                                                    |
| $\text{DELETE(key)}$<br>: Remove a key value pair                                                                                                                                                                          | $\text{Availabillity}=\frac{\text{up-time}}{\text{up-time}+\text{down-time}}$ ex.$99.99\% \;:<53$ min per year  |
| $\text{TTL \& automatic eviction}$<br>**ttl-based**: eviction depends on $\text{ttl}$<br>(Lazy-expiration &active expiration)<br>**memory-presure** : when memory full, system depends eviction-policy to eviction(ex.LRU) | $\text{Reliability} = P(\{\text{system non-failure in time }\; t\})$<br>: the probability of system non-failure |

### Data storage
In node:
- LRU structure: Hash map + Doubly Linkedlist
	- get/put:$O(1)$
	- Eviction: $O(1)$
		1. Capacity reached, look $tail:\text{deque.peekLast()}$
		2. unlink $tail$ node(node C): $O(1)$
		3. $\text{map.remove('key C')}$
![[Pasted image 20260807160514.png|381]]
