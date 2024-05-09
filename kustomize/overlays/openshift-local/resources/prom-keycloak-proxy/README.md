# OpenShift Local deployment of prom-keycloak-proxy

## Create the prom-keycloak-proxy-secrets Secret

Obtain the NERC realm nerc client secret to complete the literals in this command.

```bash
oc -n prom-keycloak-proxy create secret generic prom-keycloak-proxy-secrets \
  --from-literal=PROXY_PROMETHEUS_BASE_URL=https://observatorium-api-open-cluster-management-observability.apps.example.com/api/metrics/v1/default \
  --from-literal=PROXY_AUTH_CLIENT_ID=nerc \
  --from-literal=PROXY_AUTH_CLIENT_SECRET=
```

## Create the prom-keycloak-proxy-certs Secret

Obtain the prometheus TLS certificate, TLS key, and CA certificate.

```bash
oc -n open-cluster-management-observability extract secret/observability-grafana-certs --keys=tls.crt --to=$HOME/Downloads/
oc -n open-cluster-management-observability extract secret/observability-grafana-certs --keys=tls.key --to=$HOME/Downloads/
oc -n open-cluster-management-observability extract secret/observability-server-ca-certs --keys=ca.crt --to=$HOME/Downloads/
```

Create the prom-keycloak-proxy-certs Secret.

```bash
oc -n prom-keycloak-proxy create secret generic prom-keycloak-proxy-certs \
  --from-file=$HOME/Downloads/tls.crt \
  --from-file=$HOME/Downloads/tls.key \
  --from-file=$HOME/Downloads/ca.crt
```

