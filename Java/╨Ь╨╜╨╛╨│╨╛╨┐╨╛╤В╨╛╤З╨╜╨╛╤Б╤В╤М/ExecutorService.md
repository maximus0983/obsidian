Runnable:
- **Не возвращает значения** и **не бросает проверяемых исключений**.
- Используется, когда нужно просто выполнить код "в фоне" без результата.
  
Callable:
-  **Возвращает значение** (тип `V`) и может **бросать проверяемые исключения**.
- Чаще всего используется вместе с `ExecutorService` и `Future`.
  
  
ExecutorService` универсален: он умеет и `Runnable`, и `Callable`.  
Просто с `Runnable` у тебя максимум — `null` в `Future`, а с `Callable` — полноценный результат или исключение.
```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(task);
Integer result = future.get();
executor.shutdown();
```


![[Снимок экрана 2025-10-26 в 15.47.30.png]]