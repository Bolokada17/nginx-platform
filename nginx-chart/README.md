# NGINX Helm Chart

Dieses Chart deployt drei NGINX-Replikas mit Rolling Updates, Readiness-/Liveness-Probes, Ressourcenlimits und Standard-Logging nach stdout/stderr. Der NGINX Exporter läuft als Sidecar und stellt Metriken auf Port `9113` bereit.

## Installation

```bash
kubectl create namespace nginx-prod
helm install nginx ./nginx-chart -n nginx-prod
```

Alternativ legt Helm den Namespace automatisch an:

```bash
helm install nginx ./nginx-chart -n nginx-prod --create-namespace
```

Der Service ist standardmäßig `ClusterIP`: Der interne Service wird nicht direkt ins Internet exponiert; externer HTTP(S)-Zugriff erfolgt über den konfigurierten Ingress. Für Umgebungen ohne Ingress kann `--set service.type=LoadBalancer` verwendet werden.

## Konfiguration

Die wichtigsten Einstellungen stehen in `values.yaml`. TLS wird beispielsweise aktiviert mit:

```yaml
ingress:
  tls:
    - secretName: nginx-tls
      hosts:
        - nginx.example.com
```

Prometheus-kompatible Scrape-Annotations werden standardmäßig auf den Pod gesetzt. Das importierbare Grafana-Dashboard liegt unter `grafana/nginx-dashboard.json`. Optionale Prometheus Operator Alerts werden mit `--set alerts.enabled=true` aktiviert und benötigen die CRD `PrometheusRule`.

Die Standardmetriken des NGINX Exporters enthalten Requests und Verbindungsstatus. Die Latenz- und Statuscode-Panels im Dashboard verwenden die üblichen `nginx_ingress_controller_*`-Metriken des NGINX Ingress Controllers; dafür muss dieser ebenfalls von Prometheus gescrapt werden. In einer Installation ohne Ingress-Controller können diese beiden Panels auf die jeweils vorhandene Metrikquelle angepasst werden.

## Argo CD

`argocd/application.yaml` definiert eine Argo-CD-Application, die dieses Verzeichnis als Helm-Chart deployt. Vor dem Anwenden müssen `repoURL` und bei Bedarf `targetRevision` an das eigene Git-Repository angepasst werden:

```bash
kubectl apply -f ./nginx-chart/argocd/application.yaml
```

Die Application muss im Argo-CD-Namespace (`argocd`) angelegt werden. Argo CD verwendet dabei `values.yaml`, erstellt den Ziel-Namespace `nginx-prod` automatisch und synchronisiert Änderungen aus dem Branch `main`. Für eine manuelle Synchronisation kann der Block `syncPolicy.automated` entfernt werden.

## Struktur

```text
nginx-chart/
├── Chart.yaml
├── values.yaml
├── README.md
├── grafana/nginx-dashboard.json
├── argocd/application.yaml
├── .github/workflows/deploy.yml
└── templates/
    ├── _helpers.tpl
    ├── namespace.yaml
    ├── configmap.yaml
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── prometheus-rules.yaml
```
