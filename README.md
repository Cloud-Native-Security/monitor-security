# Monitor your cluster security

This repository uses the following applications:
- [Prometheus Stack Helm Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
- [Grafana](https://grafana.com/)
- [Promtail & Loki](https://grafana.com/oss/loki/)
- [Trivy Exporter](https://github.com/giantswarm/starboard-exporter)
- [Trivy Helm Chart](https://github.com/aquasecurity/trivy-operator)
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
helm repo update
```

```
helm upgrade --install prom prometheus-community/kube-prometheus-stack -n monitoring --values observability-conf/prom-values.yaml
```

Install promtail to collect logs from every node:

```
helm repo add grafana https://grafana.github.io/helm-charts
```


```
helm upgrade --install promtail grafana/promtail -f observability-conf/promtail-values.yaml -n monitoring
```

Install loki to collect all the logs from promtail:
```
helm upgrade --install loki grafana/loki-distributed -n monitoring
```

Install Trivy operator:
```
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm repo update
```

And finally, the Helm chart can be installed with the following command:

```
helm upgrade --install trivy-operator aqua/trivy-operator \
  --namespace trivy-system \
  --create-namespace \
  --set="trivy.ignoreUnfixed=true" \
  --set="serviceMonitor.enabled=true" \
  --version 0.18.0
```

Alternatively, it's also possible to set a custom values.yaml manifest that overrides the default values in the Helm Chart. We have set up the following [values.yaml](./observability-conf/trivy-values.yaml) manifest for the Trivy Operator. To provide the file upon installing the operator, use the following command:
```
helm upgrade --install trivy-operator aqua/trivy-operator \
  --namespace trivy-system \
  --create-namespace \
  --values ./observability-conf/trivy-values.yaml \
  --version 0.18.0
```

Make sure to cross-check the updated installation incl. the latest versio  of the operator in the docs: https://aquasecurity.github.io/trivy-operator/latest/operator/installation/helm/

## Open the dashboards in Grafana

You can then port-forward to grafana:
```
kubectl port-forward service/prom-grafana -n monitoring 3000:80
```

The login is:
    Username: admin
    Password: prom-operator

And provide Grafana with the dashboards in the [observability-conf](./observability-conf/) folder.
Note that Trivy also has a custom Dashboard -- [the ID: 17813 ]

![Vulnerability stats](./assets/vulnerabilities.png)

![Tracee logs](./assets/traceelogs.png)
