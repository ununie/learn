# prometheus函数
## 参考
> https://qingwave.github.io/prometheus-best-practice-operation/#%E6%95%B0%E6%8D%AE%E5%87%86%E7%A1%AE%E6%80%A7
> 
> increase原理
> 
> https://ihac.xyz/2018/12/11/Prometheus-Extrapolation原理解析/ 
> 
> 一些常用的指标
> 
> http://devgou.com/article/Prometheus/
> 
> 监控4个指标
> 
> https://juejin.cn/post/6995356851084722212 
>

---
## 一些函数

`rate`
> prometheus中rate只能用于counter类型，对于需要聚合的数据需要先rate再sum，而不是rate(sum)
> 
> rate/increase/delta等操作对于原始值进行了外推（类似线性插件），得到的不是准确值。如rate(http_requests_total[2m])指两分钟内每秒平均请求量，通过2m内首尾两个数据外推得到差值，比120s得到； 同理increase(http_requests_total[2m])指的不是首尾两个值的增长量，而是外推后计算出2m内的增长量。

`predict_linear`

线性回归预测，适合线性数据的预测，如预测etcd的未来4小时文件描述符使用量。
```bash
predict_linear(cluster:etcd:fd_utilization[1h], 3600 * 4)
```
`quantile_over_time`
一段时间内统计分位数
```bash
 # 一天内请求量的90分位
quantile_over_time(0.9, http_requests_total[1d])
```
---
## 常用的指标
```bash
#springboot指标
# 慢请求
sum(increase(http_server_requests_seconds_sum{app="$app", exception="None",status!~"5.."}[2m])) by (uri)/sum(increase(http_server_requests_seconds_count{app="$app",exception="None", status!~"5.."}[2m])) by (uri) >0.5
# 单条线的平均 avg_over_time
count(avg_over_time(backtest_strategy_summary{application="backtest"}[1h])>1000)
```

