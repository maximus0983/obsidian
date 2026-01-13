[[Метрики]]

 Все лейблы вместе описывают собой time series (временной ряд), т.е. это как бы **имя таблицы**, составленное из всех key-value. В этом ряду лежат значения `[(timestamp1, double1), (timestamp2, double2), ...]`
 ![[Pasted image 20230402215954.png]]


Например, мы пишем метрику запросов с тегами `app, instance, datacenter, region`. Выведем по графику на каждый `app+instance`
```
sum (http_requests_total) by (app, instance)
```
Можно группировать по «всему кроме тега»:
```
sum (http_requests_total) without (instance)
```
