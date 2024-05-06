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

## Update fields of the keycloak-permissions-operator/roles/crd-vars/vars/KeycloakAuthorization.yaml

```bash
ansible-playbook create-crd.yaml -e CRD_NAME=KeycloakAuthorization
```

## Deploy a sample KeycloakAuthorization

```bash
ansible-playbook apply-keycloakauthorization.yaml \
  -e ansible_operator_meta_namespace=keycloak \
  -e AUTH_BASE_URL=https://keycloak.apps-crc.testing \
  -e AUTH_USERNAME=admin \
  -e AUTH_PASSWORD=$(oc -n keycloak get secret/keycloak-initial-admin -o jsonpath={.data.password} | base64 -d) \
  -e crd_path=kustomize/overlays/openshift-local/resources/keycloak/keycloakauthorizations/nerc/keycloakauthorization.yaml
```
