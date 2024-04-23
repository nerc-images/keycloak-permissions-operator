# keycloak-permissions-operator
An OpenShift Operator for managing Keycloak resources, scopes, policies, and permissions for fine-grained resource permissions. 

## Build and deploy the operator to an OpenShift cluster

```bash
cd keycloak-permissions-operator
make podman-build podman-push deploy
```

## Check the logs of the operator

```bash
oc -n keycloak-permissions-operator-system logs -l control-plane=controller-manager -f
```
