
# Contribute the latest operator version to Kubernetes

Start by deleting the previous operator bundle in your OpenShift Local cluster.

```bash
oc -n keycloak delete catalogsource/keycloak-permissions-operator-catalog
oc -n keycloak delete subscription -l operators.coreos.com/keycloak-permissions-operator.keycloak=''
oc -n keycloak delete csv -l operators.coreos.com/keycloak-permissions-operator.keycloak=''
```

## Set up reusable Operator Bundle environment variables

```bash
export USERNAME=computate
export VERSION=1.4.1
export IMG=quay.io/nerc-images/keycloak-permissions-operator:$VERSION
export BUNDLE_IMG=quay.io/nerc-images/keycloak-permissions-operator-bundle:$VERSION
```

## Update the Operator version in the MakeFile and ClusterServiceVersion

```bash
perl -pi -e 's/^(VERSION \?= ).*/${1}'"$VERSION"'/g' Makefile
perl -pi -e 's/(containerImage: quay\.io\/nerc-images\/keycloak-permissions-operator:).*/${1}'"$VERSION"'/g' \
  config/manifests/bases/keycloak-permissions-operator.clusterserviceversion.yaml
```

## Build and validate the latest Operator Bundle

```bash
make podman-build podman-push
make bundle
make bundle-build bundle-push
operator-sdk bundle validate $BUNDLE_IMG
```

## Deploy the latest Operator bundle to OpenShift Local cluster for testing

```bash
operator-sdk -n keycloak run bundle $BUNDLE_IMG --security-context-config restricted
```
## Contribute latest Operator version to operatorhub.io

### Fork the k8s-operatorhub/community-operators repo

- Create a fork of the community-operators repo: https://github.com/k8s-operatorhub/community-operators

### Clone the k8s-operatorhub/community-operators repo

- Clone your community-operators fork repository.

```bash
git clone git@github.com:computate/community-operators.git ~/.local/src/community-operators/
```

### Set up the community-operators upstream remote

```bash
cd ~/.local/src/community-operators/
git remote add upstream git@github.com:k8s-operatorhub/community-operators.git
```

### Create a new branch for your version

```bash
git checkout -b keycloak-permissions-operator-$VERSION
```

### Rsync latest version of the operator to community-operators

```bash
mkdir -p ~/.local/src/community-operators/operators/keycloak-permissions-operator/$VERSION/
rsync -r bundle/ ~/.local/src/community-operators/operators/keycloak-permissions-operator/$VERSION/
```

### Commit your changes

- Git add and git commit your changes. 
- Be sure to include your signature: `Signed-off-by: My Name <me@example.com>`
- Make a pull request with the gh CLI.

### Contribute the operator to operatorhub.io

```bash
gh pr create -f
```
