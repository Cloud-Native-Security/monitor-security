# Monitor your cluster security

This repository uses the following applications:
- [Prometheus Stack Helm Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
- [Grafana](https://grafana.com/)
- [Promtail & Loki](https://grafana.com/oss/loki/)
- [Starboard Exporter](https://github.com/giantswarm/starboard-exporter)
- [Starboard Helm Chart](https://github.com/aquasecurity/starboard)
- [Tracee](https://github.com/aquasecurity/tracee)

Here is how to use the resources:

Create a monitoring namespace:
```
kubectl create ns monitoring
```

Install the helm prometheus stack chart:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```
helm upgrade --install prom prometheus-community/kube-prometheus-stack -n monitoring --values observability-conf/prom-values.yaml
```

Install promtail to colelct logs from every node:


```
helm upgrade --install promtail grafana/promtail -f observability-conf/promtail-values.yaml -n monitoring
```

Install loki to collect all the logs from promtail:
```
helm upgrade --install loki grafana/loki-distributed -n monitoring
```

Install Starbaord operator:
```
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm repo update
```

```
helm install starboard-operator aqua/starboard-operator \
  --namespace starboard-system \
  --create-namespace \
  --set="trivy.ignoreUnfixed=true" \
  --version 0.10.4
```

Install the Starboard exporter:

```
helm repo add giantswarm https://giantswarm.github.io/giantswarm-catalog
helm repo update
helm upgrade -i starboard-exporter --namespace <starboard namespace> giantswarm/starboard-exporter
```

Install tracee to monitor your cluster:

```
kubectl apply -f observability-conf/tracee.yaml
```

Create application:
```
kubectl create ns app
kubectl apply -f app-manifests -n app
```

## Open the dashboards in Grafana

You can then port-forward to grafana:
```
kubectl port-forward service/prom-grafana -n monitoring 3000:80
```

The login is:
    Username: admin
    Password: prom-operator

And provide Grafana with the dashboards in the [observability-conf](./observability-conf/) folder.

![Vulnerability stats](./assets/vulnerabilities.png)

![Tracee logs](./assets/traceelogs.png)
