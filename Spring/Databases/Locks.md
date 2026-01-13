
<iframe width="560" height="315" src="https://www.youtube.com/embed/nuBi2XbHH18?si=Ru_i20y-SXB68iH6" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


я так понял exclusive lock - это когда никто другой не может даже читать это значение. а shared lock - это когда никто другой не может изменять значение. при shared lock к примеру операция update блокируется и ждет, пока он снимется. 
и вроде как ограничение такое есть, что когда кто-то поставил exclusive lock, то никто не может поставить shared lock туда же, и наоборот, если кто-то поставил shared lock, другие не могут поставить exclusive lock на эту же строку





When dealing with conflicts, you have two options:
- You can try to avoid the conflict, and that's what Pessimistic Locking does.
- Or, you could allow the conflict to occur, but you need to detect it upon committing your transactions, and that's what Optimistic Locking does.
## Pessimistic Locking

Pessimistic locking achieves this goal by taking a shared or read lock on the account so Bob is prevented from changing the account.

[![Lost Update Pessimistic Locking](https://i.stack.imgur.com/oybRy.png)](https://vladmihalcea.com/optimistic-vs-pessimistic-locking/)

In the diagram above, both Alice and Bob will acquire a read lock on the `account` table row that both users have read. The database acquires these locks on SQL Server when using Repeatable Read or Serializable.

Because both Alice and Bob have read the `account` with the PK value of `1`, neither of them can change it until one user releases the read lock. This is because a write operation requires a write/exclusive lock acquisition, and shared/read locks prevent write/exclusive locks.

Only after Alice has committed her transaction and the read lock was released on the `account` row, Bob `UPDATE` will resume and apply the change. Until Alice releases the read lock, Bob's UPDATE blocks.




## Optimistic Locking

Optimistic Locking allows the conflict to occur but detects it upon applying Alice's UPDATE as the version has changed.

[![Application-level transactions](https://i.stack.imgur.com/DEdlF.png)](https://vladmihalcea.com/optimistic-vs-pessimistic-locking/)

This time, we have an additional `version` column. The `version` column is incremented every time an UPDATE or DELETE is executed, and it is also used in the WHERE clause of the UPDATE and DELETE statements. For this to work, we need to issue the SELECT and read the current `version` prior to executing the UPDATE or DELETE, as otherwise, we would not know what version value to pass to the WHERE clause or to increment.



## Application-level transactions

Relational database systems have emerged in the late 70's early 80's when a client would, typically, connect to a mainframe via a terminal. That's why we still see database systems define terms such as SESSION setting.

Nowadays, over the Internet, we no longer execute reads and writes in the context of the same database transaction, and ACID is no longer sufficient.

For instance, consider the following use case:

[![Application-level transactions and Optimistic Locking](https://i.stack.imgur.com/FCyHh.png)](https://vladmihalcea.com/preventing-lost-updates-in-long-conversations/)

Without optimistic locking, there is no way this Lost Update would have been caught even if the database transactions used Serializable. This is because reads and writes are executed in separate HTTP requests, hence on different database transactions.

So, optimistic locking can help you prevent Lost Updates even when using application-level transactions that incorporate the user-think time as well.





С блокировками в postgres ситуация такова, что описанные выше exclusive lock  и shared lock редко используются в таком виде, а в большинстве случаев там используются более легковесные блокировки. самое важное различие в блокировках в том, какие из них совместимы между собой, а какие конфликтуют. это можно посмотреть в соответствующих таблицах в документации. для собеса скорее всего будет достаточно запомнить самую агрессивную блокировку и самую лайтовую ( и для table-level, и для row-level)
https://www.postgresql.org/docs/current/explicit-locking.html




13.3.4. Deadlocks 
The use of explicit locking can increase the likelihood of deadlocks, wherein two (or more) transactions each hold locks that the other wants. For example, if transaction 1 acquires an exclusive lock on table A and then tries to acquire an exclusive lock on table B, while transaction 2 has already exclusive-locked table B and now wants an exclusive lock on table A, then neither one can proceed. PostgreSQL automatically detects deadlock situations and resolves them by aborting one of the transactions involved, allowing the other(s) to complete. (Exactly which transaction will be aborted is difficult to predict and should not be relied upon.)

Note that deadlocks can also occur as the result of row-level locks (and thus, they can occur even if explicit locking is not used). Consider the case in which two concurrent transactions modify a table. The first transaction executes:

UPDATE accounts SET balance = balance + 100.00 WHERE acctnum = 11111;
This acquires a row-level lock on the row with the specified account number. Then, the second transaction executes:

UPDATE accounts SET balance = balance + 100.00 WHERE acctnum = 22222;
UPDATE accounts SET balance = balance - 100.00 WHERE acctnum = 11111;
The first UPDATE statement successfully acquires a row-level lock on the specified row, so it succeeds in updating that row. However, the second UPDATE statement finds that the row it is attempting to update has already been locked, so it waits for the transaction that acquired the lock to complete. Transaction two is now waiting on transaction one to complete before it continues execution. Now, transaction one executes:

UPDATE accounts SET balance = balance - 100.00 WHERE acctnum = 22222;
Transaction one attempts to acquire a row-level lock on the specified row, but it cannot: transaction two already holds such a lock. So it waits for transaction two to complete. Thus, transaction one is blocked on transaction two, and transaction two is blocked on transaction one: a deadlock condition. PostgreSQL will detect this situation and abort one of the transactions.

The best defense against deadlocks is generally to avoid them by being certain that all applications using a database acquire locks on multiple objects in a consistent order. In the example above, if both transactions had updated the rows in the same order, no deadlock would have occurred. One should also ensure that the first lock acquired on an object in a transaction is the most restrictive mode that will be needed for that object. If it is not feasible to verify this in advance, then deadlocks can be handled on-the-fly by retrying transactions that abort due to deadlocks.

So long as no deadlock situation is detected, a transaction seeking either a table-level or row-level lock will wait indefinitely for conflicting locks to be released. This means it is a bad idea for applications to hold transactions open for long periods of time (e.g., while waiting for user input).




13.3.5. Advisory Locks 
PostgreSQL provides a means for creating locks that have application-defined meanings. These are called advisory locks, because the system does not enforce their use — it is up to the application to use them correctly. Advisory locks can be useful for locking strategies that are an awkward fit for the MVCC model. For example, a common use of advisory locks is to emulate pessimistic locking strategies typical of so-called “flat file” data management systems. While a flag stored in a table could be used for the same purpose, advisory locks are faster, avoid table bloat, and are automatically cleaned up by the server at the end of the session.

Advisory Locks  - нужны, когда нам нужны более верхнеуровнево управлять ( вне транзакций)



<iframe width="560" height="315" src="https://www.youtube.com/embed/URwmzTeuHdk?si=thiZ347ywWaDjgrb" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
