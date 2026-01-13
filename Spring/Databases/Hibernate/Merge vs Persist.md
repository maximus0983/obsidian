Короче говоря при merge возвращается КОПИЯ обьекта, и вот она уже кладется в PersistenceContext.

Either way will add an entity to a PersistenceContext, the difference is in what you do with the entity afterwards.

Persist takes an entity instance, adds it to the context and makes that instance managed (i.e. future updates to the entity will be tracked).

Merge returns the managed instance that the state was merged with. It does return something that exists in PersistenceContext or creates a new instance of your entity. In any case, it will copy the state from the supplied entity, and return a managed copy. The instance you pass in will not be managed (any changes you make will not be part of the transaction - unless you call merge again). Though you can use the returned instance (managed one).

Maybe a code example will help.

```java
MyEntity e = new MyEntity();

// scenario 1
// tran starts
em.persist(e); 
e.setSomeField(someValue); 
// tran ends, and the row for someField is updated in the database

// scenario 2
// tran starts
e = new MyEntity();
em.merge(e);
e.setSomeField(anotherValue); 
// tran ends but the row for someField is not updated in the database
// (you made the changes *after* merging)
      
// scenario 3
// tran starts
e = new MyEntity();
MyEntity e2 = em.merge(e);
e2.setSomeField(anotherValue); 
// tran ends and the row for someField is updated
// (the changes were made to e2, not e)
```

Scenario 1 and 3 are roughly equivalent, but there are some situations where you'd want to use Scenario 2.