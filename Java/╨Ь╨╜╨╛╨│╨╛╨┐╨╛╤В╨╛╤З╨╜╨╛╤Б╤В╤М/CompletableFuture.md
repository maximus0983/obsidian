
![[Снимок экрана 2025-10-26 в 14.20.23 1.png]]


```java
CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> {
    System.out.println("Фоновая задача: " + Thread.currentThread().getName());
});

CompletableFuture<String> futur2 = CompletableFuture.supplyAsync(()> {
    return "Результат";
});
```

**Комбинирование нескольких задач**
```java
CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(() -> 20);

f1.thenCombine(f2, Integer::sum)
  .thenAccept(System.out::println); // 30

```

`allOf` — ждет завершения всех
```java
CompletableFuture<Void> all = CompletableFuture.allOf(f1, f2);
```


```java
request 
.thenApply(res -> "Ответ: " + res)// не будет вызван, если входящий Future уже в состоянии ошибки
.exceptionally(ex -> "Фолбэк: " + ex.getMessage()) // сработает только при ошибке. Это значение станет **новым результатом future**, который пойдёт дальше.
.thenAccept(System.out::println);
```
- `exceptionally` всегда «чинит» цепочку: если он вернул значение, дальнейшие стадии считают future успешным.
- Если же внутри `exceptionally` снова кинуть ошибку — дальше цепочка останется в состоянии exceptional.



Методы `thenApply` выполняются в том же потоке, что и завершившаяся задача, а `thenApplyAsync` — в пуле потоков (`ForkJoinPool.commonPool()` по умолчанию  или в  твоём `Executor`).

Важно: если ты начнёшь с `runAsync`, то в первой функции (`thenApply`, `thenAccept`, `thenRun`) **аргумент всегда будет `null`**, потому что у `Runnable` нет возвращаемого значения.

Если нужен результат → используй **`supplyAsync(...)`**.
