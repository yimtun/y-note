## prometheus



```
wget  https://github.com/prometheus/prometheus/releases/download/v2.37.0-rc.1/prometheus-2.37.0-rc.1.darwin-amd64.tar.gz
```



```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.darwin-amd64.tar.gz
```



```
./prometheus  --web.enable-lifecycle --config.file ./prometheus.yml 
```



```
http://127.0.0.1:9090/
```



```
http://127.0.0.1:9090/targets
```





### PQL



```
```



```
avg(node_load5{instance="127.0.0.1:9100",job="node-exporter"}) / count(count(node_cpu_seconds_total{instance="127.0.0.1:9100",job="node-exporter"}) by (cpu)) * 100
```



```
avg_over_time(node_load5{instance="127.0.0.1:9100",job="node-exporter"}[10s])
```



```
node_load5{instance="127.0.0.1:9100",job="node-exporter"}[10m]
```





## grafana



```
wget  https://dl.grafana.com/oss/release/grafana-9.0.2.darwin-amd64.tar.gz
```



```
./bin/grafana-server 
```



```
http://127.0.0.1:3000/login
```



```
admin
admin
```





## node-exporter





```
curl -X POST http://127.0.0.1:9090/-/reload
```



configfile



```
  - job_name: 'node-exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['127.0.0.1:9100' ]
        labels:
          L1: "test"
          L2: "red"
```









