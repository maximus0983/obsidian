![[Снимок экрана 2023-07-21 в 21.58.07.png]]


![[Снимок экрана 2023-07-21 в 21.58.41.png]]

Postgres - READ_COMMITED

![[Снимок экрана 2023-07-21 в 21.59.06.png]]


![[Снимок экрана 2023-07-21 в 22.18.57.png]]

![[Снимок экрана 2023-07-21 в 22.19.22.png]]

PostgreSQL does not support dirty reads (`READ UNCOMMITTED`). As @a_horse_with_no_name pointed out, [the manual](http://www.postgresql.org/docs/current/static/sql-set-transaction.html) says:

> The SQL standard defines one additional level, `READ UNCOMMITTED`. In PostgreSQL `READ UNCOMMITTED` is treated as `READ COMMITTED`.
> 

![[Снимок экрана 2023-07-21 в 22.19.42.png]]

Когда ставим уровень изоляции REPEATABLE READ, то  даже если  другая транзакция что-то там поменять успела параллельно, все равно внутри транзакции с REPEATABLE READ селекты будут брать то значение, которое было в момент старта транзакции с REPEATABLE READ.

Applications using this level must be prepared to retry transactions due to serialization failures.

`UPDATE`, `DELETE`, `MERGE`, `SELECT FOR UPDATE`, and `SELECT FOR SHARE` commands behave the same as `SELECT` in terms of searching for target rows: they will only find target rows that were committed as of the transaction start time. However, such a target row might have already been updated (or deleted or locked) by another concurrent transaction by the time it is found. In this case, the repeatable read transaction will wait for the first updating transaction to commit or roll back (if it is still in progress). If the first updater rolls back, then its effects are negated and the repeatable read transaction can proceed with updating the originally found row. But if the first updater commits (and actually updated or deleted the row, not just locked it) then the repeatable read transaction will be rolled back with the message

ERROR:  could not serialize access due to concurrent update

because a repeatable read transaction cannot modify or lock rows changed by other transactions after the repeatable read transaction began.

When an application receives this error message, it should abort the current transaction and retry the whole transaction from the beginning. The second time through, the transaction will see the previously-committed change as part of its initial view of the database, so there is no logical conflict in using the new version of the row as the starting point for the new transaction's update.

Note that only updating transactions might need to be retried; read-only transactions will never have serialization conflicts.

The Repeatable Read mode provides a rigorous guarantee that each transaction sees a completely stable view of the database. However, this view will not necessarily always be consistent with some serial (one at a time) execution of concurrent transactions of the same level. For example, even a read-only transaction at this level may see a control record updated to show that a batch has been completed but _not_ see one of the detail records which is logically part of the batch because it read an earlier revision of the control record. Attempts to enforce business rules by transactions running at this isolation level are not likely to work correctly without careful use of explicit locks to block conflicting transactions.




![[Снимок экрана 2023-07-21 в 22.20.23.png]]

Фантомное, это когда не одна конкретная запись, а именно результирующий set может быть разным.
В Postgresql  уровень изоляции REPEATABLE_READ предотвращает и фантомные чтения в том числе. Но не предотвращает Serialization Anomaly.
Что такое  Serialization Anomaly :
The result of successfully committing a group of transactions is inconsistent with all possible orderings of running those transactions one at a time.




Serializable transactions take special "SI" locks which track read and write access and survive a commit. They don't block other sessions, but are used to determine if there _may_ be a conflict. Serializable isolation level only works properly if _all_ concurrent transactions use the serializable isolation level.