# keycloak-permissions-operator
An OpenShift Operator for managing Keycloak resources, scopes, policies, and permissions for fine-grained resource permissions.

## Install the Operator SDK

```bash
mkdir -p ~/.local/opt
export ARCH=$(case $(uname -m) in x86_64) echo -n amd64 ;; aarch64) echo -n arm64 ;; *) echo -n $(uname -m) ;; esac)
export OS=$(uname | awk '{print tolower($0)}')
export OPERATOR_SDK_DL_URL=https://github.com/operator-framework/operator-sdk/releases/download/v1.28.1
curl -o ~/.local/opt/operator-sdk_${OS}_${ARCH} -LO ${OPERATOR_SDK_DL_URL}/operator-sdk_${OS}_${ARCH}
gpg --keyserver keyserver.ubuntu.com --recv-keys 052996E2A20B5C7E curl -o ~/.local/opt/operator-sdk-checksums.txt -LO ${OPERATOR_SDK_DL_URL}/checksums.txt
curl -o ~/.local/opt/operator-sdk-checksums.txt.asc -LO ${OPERATOR_SDK_DL_URL}/checksums.txt.asc
gpg -u "Operator SDK (release) <cncf-operator-sdk@cncf.io>" --verify ~/.local/opt/operator-sdk-checksums.txt.asc
grep operator-sdk_${OS}_${ARCH} ~/.local/opt/operator-sdk-checksums.txt | sha256sum -c -
mkdir -p ~/.local/bin
chmod +x ~/.local/opt/operator-sdk_${OS}_${ARCH}
mv ~/.local/opt/operator-sdk_${OS}_${ARCH} ~/.local/bin/operator-sdk
rm ~/.local/opt/operator-sdk-checksums*
```

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

## Build the latest Operator Bundle

Start by deleting the previous operator bundle in your OpenShift Local cluster.

```bash
oc -n keycloak delete catalogsource/keycloak-permissions-operator-catalog
oc -n keycloak delete subscription -l operators.coreos.com/keycloak-permissions-operator.keycloak=''
oc -n keycloak delete csv -l operators.coreos.com/keycloak-permissions-operator.keycloak=''
```

Set up some reusable environment variables for the Operator Bundle.

```bash
export USERNAME=computate
export VERSION=1.4.1
export IMG=quay.io/nerc-images/keycloak-permissions-operator:$VERSION
export BUNDLE_IMG=quay.io/nerc-images/keycloak-permissions-operator-bundle:$VERSION
```

Update the Operator version in the MakeFile and ClusterServiceVersion.

```bash
perl -pi -e 's/^(VERSION \?= ).*/${1}'"$VERSION"'/g' Makefile
perl -pi -e 's/(containerImage: quay\.io\/nerc-images\/keycloak-permissions-operator:).*/${1}'"$VERSION"'/g' \
  config/manifests/bases/keycloak-permissions-operator.clusterserviceversion.yaml
```

Now build and validate the latest Operator Bundle.

```bash
make podman-build podman-push
make bundle
make bundle-build bundle-push
operator-sdk bundle validate $BUNDLE_IMG
```

Deploy the latest Operator bundle to your local OpenShift Local cluster.

```bash
operator-sdk -n keycloak run bundle $BUNDLE_IMG --security-context-config restricted
```

## Contribute the operator to operatorhub.io

Create a fork of the community-operators repo: https://github.com/k8s-operatorhub/community-operators
Clone your community-operators fork repository.

```bash
git clone git@github.com:computate/community-operators.git ~/.local/src/community-operators/
cd ~/.local/src/community-operators/
```

Set up the community-operators upstream remote.

```bash
cd ~/.local/src/community-operators/
git remote add upstream git@github.com:k8s-operatorhub/community-operators.git
```

Create a new branch for your version.

```bash
git checkout -b keycloak-permissions-operator-$VERSION
```

Copy the latest version of the operator to community-operators.

```bash
mkdir -p ~/.local/src/community-operators/operators/keycloak-permissions-operator/$VERSION/
rsync -r bundle/ ~/.local/src/community-operators/operators/keycloak-permissions-operator/$VERSION/
```

- Git add and git commit your changes. 
- Be sure to include your signature: `Signed-off-by: My Name <me@example.com>`
- Make a pull request with the gh CLI.

```bash
gh pr create -f
```

## Contributing new versions of the operator

- Learn [how to contribute new versions of the operator to Kubernetes operatorhub.io](doc/contribute-operator-to-kubernetes.md)
- Learn [how to contribute new versions of the operator to Red Hat OpenShift](doc/contribute-operator-to-openshift.md)
