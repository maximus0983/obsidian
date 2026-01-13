[[Метрики]]

Let’s say we want to find out how many requests our service is handling right now. Our metric `http_requests_total{code="200",handler="/api/v1/query"}` is an instant vector with values that represent a monotonically increasing counter. This counter measures the number of requests our service has received in total. - Instant Vector - общее количество запросов
Если используем к этой метрике http_requests_total[2m] то получим Range Vector
[f(x-30s), f(x-60s), f(x-90s),f(x-120s)] - общее время в [] / время скрепинга - количество элементов в массиве range vector

Применив аггрегирующую функцию - получим нужное значение.
increase: разница между первым и последним значением
rate: разница между первым и последним значением деленная на время (получается изменение в секунду - производная)