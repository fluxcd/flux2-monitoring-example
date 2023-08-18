# flux2-monitoring-example

This repository is an example of how to make use of
[kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
and
[loki-stack](https://github.com/grafana/helm-charts/tree/main/charts/loki-stack)
to monitor Flux.

Components:

* **kube-state-metrics** - generates metrics about the state of the Flux objects
* **Prometheus Operator** - manages Prometheus clusters atop Kubernetes
* **Prometheus** - collects and stores metrics from the Flux controllers and kube-state-metrics
* **Promtail** - collects the logs from the Flux controllers
* **Loki** - stores the logs collected by Promtail
* **Grafana** dashboards - displays the Flux control plane resource usage, reconciliation stats and logs

## Quickstart

### Create a Kubernetes cluster

For a quick local test, you can use [Kubernetes kind](https://kind.sigs.k8s.io/docs/user/quick-start/).
Any other Kubernetes setup will work as well though.

Create a cluster called `test` with the kind CLI:

```shell
kind create cluster --name test
```

### Fork the GitHub repository

In order to follow this guide you'll need a GitHub account and a
[personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)
that can create repositories (check all permissions under `repo`).

Add the GitHub PAT and username to your shell environment:

```sh
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>
```

Fork this repository on your personal account and clone it locally:

```shell
git clone https://github.com/${GITHUB_USER}/flux2-monitoring-example.git
cd flux2-monitoring-example
```

### Bootstrap Flux

Install the Flux controllers on the test cluster:

```shell
flux bootstrap github \
    --owner=${GITHUB_USER} \
    --repository=flux2-monitoring-example \
    --branch=main \
    --personal \
    --path=clusters/test
```

Wait for Flux to deploy the monitoring stack with:

```shell
flux get kustomizations --watch
```

After Flux has finished reconciling, you can list the pods in the monitoring namespace with:

```console
$ kubectl -n monitoring get po
NAME                                                        READY
kube-prometheus-stack-grafana-5c976ff4cf-xgmwm              3/3
kube-prometheus-stack-kube-state-metrics-5dcf4c4697-jvlvh   1/1
kube-prometheus-stack-operator-75f9fdcbf6-98zmh             1/1
kube-prometheus-stack-prometheus-node-exporter-j4vhb        1/1
loki-stack-0                                                1/1
loki-stack-promtail-dcg64                                   1/1
prometheus-kube-prometheus-stack-prometheus-0               2/2
```

### Accessing Grafana

To access Grafana, start port forward in a separate shell:

```shell
kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana  3000:80
```

Navigate to `http://localhost:3000` in your browser and login with user `admin` and password `flux`.

Flux dashboards:
- [Reconciliation stats](http://localhost:3000/d/flux-cluster/flux-cluster-stats)
- [Control plane stats](http://localhost:3000/d/flux-control-plane/flux-control-plane)
- [Control plane logs](http://localhost:3000/d/flux-logs/flux-logs)
