Install Prometheus Stack
* helm upgrade --install prometheus prometheus-community/kube-prometheus-stack --values prometheus_values.yaml  -n prometheus

#Install Gitlab Server
* helm upgrade --install gitlab gitlab/gitlab --timeout 600s  --set global.hosts.domain=gitlab.example.com \
  --set global.hosts.externalIP=10.10.10.10   --set certmanager-issuer.email=me@gitlab.example.com \
  --set postgresql.image.tag=13.6.0
