[[Метрики]]

Для измерения можно использовать quantile или формулу ниже с rate. Рекомендуется использовать формулу с rate

https://www.robustperception.io/how-does-a-prometheus-summary-work/
To calculate the average request duration during the last 5 minutes from a histogram or summary called http_request_duration_seconds, use the following expression:

  $$\frac  {rate(http\_request\_duration\_seconds\_sum[5m])}   {rate(http\_request\_duration\_seconds\_count[5m])}$$
`rate(http_request_duration_seconds_count[5m])` количество измерений за последние 5 минут в среднем за секунду(делим общее на 600)
`rate(http_request_duration_seconds_count[5m])` is the number of observations per second over the last five minutes on average, and `rate(http_request_duration_seconds_sum[5m])` is how long they took per second on average. If we divide these we get the average duration of one observation:
https://stackoverflow.com/questions/55162093/understanding-histogram-quantile-based-on-rate-in-prometheus
`rate_xxx(t) = (value_xxx[t]-value_xxx[t-5m]) / (5m*60)` is the `quantity of items` for `[t-5m, t]`

rate - показывает как изменяется значение за секунду(то есть находим разницу между (value_xxx[t] и -value_xxx[t-5m]) - видим как изменилась разница за 5 минут, и далее делим на 300 секунд - получаем на сколько в среднем изменялось значение за секунду в течение последних 5 минут

https://www.robustperception.io/how-does-a-prometheus-summary-work/

The samples with the `quantile` label are a bit more complicated, and will often not be present. These are quantiles of the observations, so the `{quantile="0.9"}` sample indicates that the 90th percentile is around 100us. What time period is this over though?
How [Prometheus client libraries generally do it](https://github.com/prometheus/client_golang/blob/fa4aa9000d2863904891d193dea354d23f3d712a/prometheus/summary.go#L135) is to keep 10 quantile objects in memory. All observation are sent to all 10 objects, each tracking observations starting 1 minute from the next, and the oldest of these will contain up to 10 minutes worth of samples.