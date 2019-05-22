# istio-benchmark

My notes on performance + scailability benchmarking for Istio. 


## Recap: Benchmarking Istio 1.1.7 

### Install Istio 

Created a K8s cluster: https://istio.io/docs/setup/kubernetes/prepare/platform-setup/gke/ 

- GKE
- 4 nodes, each with 2vcpus
- master version `1.12.7-gke.10` (default as of 5/22/19) 

Installed Istio with Helm using production-ready instructions: https://istio.io/docs/setup/kubernetes/install/helm/ 

Downloaded Istio 1.1.7 https://github.com/istio/istio/releases 

Followed instructions here for CRD install: https://istio.io/docs/setup/kubernetes/install/helm/#option-1-install-with-helm-via-helm-template 

Installed the `default` (not `demo`) version of Istio, and enabled grafana 

```
helm template ./install/kubernetes/helm/istio --name istio --namespace istio-system \
   --set prometheus.enabled=true \
   --set grafana.enabled=true  > istio.yaml

kubectl apply -f istio.yaml 
```

**Note** - Mixer pod's (`istio-telemetry`) CPU requests and limits

```
    Limits:
      cpu:     4800m  # m = thousandth of a core, so 4800 = 4.8 cores  
      memory:  4G
    Requests:
      cpu:     1
      memory:  1G
```


### Run the benchmark 

https://github.com/kinvolk/service-mesh-benchmark/blob/master/scripts/istio/benchmark.sh

```
./benchmark.sh 1 1m 10
```

where.. 

```
10 = # apps (apps = deployments, not replicas. these are separate instances of the 3-service [emojivoto](https://github.com/BuoyantIO/emojivoto) app) 

5m = # minutes to run the test 

100 = throughput (RPS) 
```

```
watch -n 1 kubectl get pods -n emojivoto
```


### My results on Istio 1.1.7 

### `benchmark.sh 1 1m 10`  (1 app, 1 minute, 10 RPS)

```
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    7.18ms
 75.000%    7.95ms
 90.000%    8.62ms
 99.000%   10.81ms
 99.900%   19.02ms
 99.990%   19.02ms
 99.999%   19.02ms
100.000%   19.02ms
```

### `benchmark.sh 1 5m 500`  (1 app, 5 minutes, 600 RPS)

