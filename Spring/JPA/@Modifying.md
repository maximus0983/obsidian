@Modifying подсказывает спринг дате, что результат этого запроса не надо будет маппить в обьекты. Если его не поставить, то Exception будет. При этом вроде не надо его ставить на name-derived методы, а только на @Query.



Unfortunately, using modifying queries leaves the underlying persistence context outdated. However, it is possible to manage this situation. That's the subject of the next section.

If our modifying query changes entities contained in the persistence context, then this context becomes outdated. One way to manage this situation is to clear the persistence context. By doing that, we make sure that the persistence context will fetch the entities from the database next time.

However, we don't have to explicitly call the clear() method on the EntityManager. We can just use the clearAutomatically property from the @Modifying annotation:

@Modifying(clearAutomatically = true)
This way, we make sure that the persistence context is cleared after our query execution.

However, if our persistence context contained unflushed changes, clearing it would mean dropping the unsaved changes. Fortunately, there's another property of the annotation we can use in this case, flushAutomatically:

@Modifying(flushAutomatically = true)
Now the EntityManager is flushed before our query is executed.

