![[Pasted image 20260930001433.png|601]]
### In - Memory

**Buffer Pool**
: used for cache **Data Pages** and **Index Pages** in memory
- **Dirty Page**: Update item in Buffer Pool, when Buffer pool had item
- **LRU Algo management**: when memory full, LRU decide  to **Evict the least used page**(out of memory) and **update to Log Buffer**
- **Parameter**
	- `innodb_buffer_pool_size`: (Default $128MB$)set Buffer Pool size
	- `innodb_buffer_pool_instance`:spilt the Buffer Pool to avoid lock contention![[IMG_3339 2.jpg|345]]
**Log Buffer**
: used for buffering the **Redo Log** in memory.Use **Append-only** sequence write-in the updating to Log Buffer.
- **Parameter**
	- `innodb_log_buffer_size`:set Log Buffer size
	- `innodb_flush_log_at_trx-commit`
		- $0$ : update per second
		- $1$: update per commit
		- $2$: each commit write-in OS Cache, synchronize to disk for each second

### On - Disk

**Redo Log**
: used **WAL**(Write Ahead Logging), mean write the update item from Log Buffer to Redo Log, then transaction is committed.
- Parameter
	- `innodb_log_file_size`(Default $48MB$)
	- `innodb_log_files_in_group`(Default $2$)
	- note: redo log size = `file_size`$\times$ `files_in_group` 

**Tablespace** is InnoDB logical storage unit
- File-Per-Table Tablespaces: for each table create the `.idb` file to store the data and index from table
	- parameter: `innodb_file_per_table = ON`
- Undo Tablespaces : used for store Undo log
	- support **Rollback** and **MVCC**
- System Tablespace: used for store **Doublewrite Buffer** and **Change Buffer**
- General Tablespaces: user `CREATE TABLESPACE` create the Share table space.
- Temporory Tablespaces: when user search massive item `GROUP BY, DISTINCT`.., item can not process in memory then will temporary store at Temporary Tablespaces.