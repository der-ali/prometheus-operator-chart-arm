# Deploy Prometheus-Operator Helm chart on RaspberryPi ARM Kubernetes
since multiple Docker images in  [prometheus-operator helm chart](https://github.com/helm/charts/tree/master/stable/prometheus-operator) and it's subcharts(kube-state-metrics,grafana) are not arm compatible, I cross-compiled these Docker images to support armv7. The table below includes the ARM uncompatible Docker images and the cross-compiled ones.

|    not ARM compatible                             |          ARM compatible                          |
|---------------------------------------------------|--------------------------------------------------|
| kiwigrid/k8s-sidecar:0.1.20                       | 4k1l/k8s-sidecar-arm:v0.1.20                     |
| quay.io/coreos/kube-state-metrics:v1.9.1          | 4k1l/kube-state-metrics-arm:v1.9.3               |
| squareup/ghostunnel:v1.4.1                        | 4k1l/ghostunnel-arm:v1.4.1                       |
| jettech/kube-webhook-certgen:v1.0.0               | jettech/kube-webhook-certgen:v1.2.0              |
| quay.io/coreos/prometheus-operator:v0.34.0        | maxromanovsky/prometheus-operator:v0.34.0        |
| quay.io/coreos/configmap-reload:v0.0.1            | jimmidyson/configmap-reload:v0.3.0               |
| quay.io/coreos/prometheus-config-reloader:v0.34.0 | maxromanovsky/prometheus-config-reloader:v0.34.0 |
| k8s.gcr.io/hyperkube:v1.12.1                      | gcr.io/google_containers/hyperkube-arm:v1.12.1   |

## USAGE

create first a namespace to deploy the helm chart in:

```bash
kubectl create namespace monitor
```

In order to deploy the helm-chart with the ARM compatible Docker images, you need to override there values. You can achieve this in two ways:

```bash
helm install prometheus-operator stable/prometheus-operator  --namespace monitor --set grafana.sidecar.image=4k1l/k8s-sidecar-arm:v0.1.20 --set kube-state-metrics.image.repository=4k1l/kube-state-metrics-arm --set kube-state-metrics.image.tag=v1.9.3 --set prometheusOperator.tlsProxy.image.repository=4k1l/ghostunnel-arm  --set prometheusOperator.admissionWebhooks.patch.image.repository=jettech/kube-webhook-certgen --set prometheusOperator.admissionWebhooks.patch.image.tag=v1.2.0 --set prometheusOperator.image.repository=maxromanovsky/prometheus-operator --set prometheusOperator.configmapReloadImage.repository=jimmidyson/configmap-reload --set prometheusOperator.configmapReloadImage.tag=v0.3.0 --set prometheusOperator.prometheusConfigReloaderImage.repository=maxromanovsky/prometheus-config-reloader --set prometheusOperator.hyperkubeImage.repository=gcr.io/google_containers/hyperkube-arm --set prometheusOperator.tlsProxy.enabled=false --set prometheusOperator.admissionWebhooks.enabled=false
```
or you can use the provided values file (prometheus-operator.values) :

```bash
helm install prometheus-operator stable/prometheus-operator  --namespace monitor --values prometheus-operator.values --set prometheusOperator.tlsProxy.enabled=false --set prometheusOperator.admissionWebhooks.enabled=false
```

To uninstall the chart:

```bash
helm uninstall --namespace monitor prometheus-operator
```

**NOTE: This setup is not meant for production usage. The Docker images used can be old or unsecure !!**

