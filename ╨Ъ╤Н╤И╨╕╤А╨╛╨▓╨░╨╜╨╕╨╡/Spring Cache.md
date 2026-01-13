
[[Кэширование]]

## Cacheable
Для создания кэша - помечаем метод findBook аннотацией ==@Cacheable("books")==
```java
@Cacheable("books")
public Book findBook(ISBN isbn){}
```
Каждый раз, когда метод findBook будет вызываться, перед этим будет проверяться кэш books

Default key generation:
-   If no params are given, return 0.
-   If only one param is given, return that instance.
-   If more the one param is given, return a key computed from the hashes of all parameters. 
Custom key generation:
```java
@Cacheable(value="books", **key="#isbn"**
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```


## CachePut

Используется для насильного обновления кэша
```java
@Cacheable(value = "users", key = "#user.name")
public User createOrReturnCached(User user){
return repository.save(user); 
}
@CachePut(value = "users", key = "#user.name") 
public User createAndRefreshCache(User user) {
return repository.save(user); 
}
```
Первый метод будет просто возвращать закэшированные значения, второй — принудительно обновлять кэш

## CacheEvict
```java
@CacheEvict("users") public void deleteAndEvict(Long id) {
repository.deleteById(id);
}
```
Иногда возникает необходимость жёстко обновить какие-то данные в кэше. Например, сущность уже удалена из базы, но она по-прежнему доступна из кэша. Для сохранения консистентности данных, нам необходимо хотя бы не хранить в кэше удалённые данные