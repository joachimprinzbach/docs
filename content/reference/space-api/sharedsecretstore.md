---
title: "SharedSecretStore"
weight: 10
---

`SharedSecretStore` represents a shared `SecretStore` projected as `ClusterSecretStore` into matching ControlPlanes in the same namespace. Once projected into a control plane, you can reference it from `ExternalSecret` instances as part of `storeRef` fields. The secret store configuration including referenced credential aren't leaked into the control planes. In that sense, they're secure as they're invisible to the control plane workloads.

An example of a `sharedsecretstore.yaml` file:

```yaml
apiVersion: spaces.upbound.io/v1alpha1
kind: SharedSecretStore
metadata:
  name: shared-vault-store
  namespace: default
spec:
  
  # The name to be used on external-secrets.io/v1beta1 SecretStores created in the control plane.
  # Optional, if not set, SharedSecretStore name will be used
  secretStoreName: "vault-store"

  # The metadata of the secret store to be created
  secretStoreMetadata:
    labels:
      acmeco/store: vault

  # The store is projected only to control planes matching the provided label selector.
  controlPlaneSelector:
    labelSelectors:
    - controlplane: dev

    # or by providing the list of controlplane names
    names:
      - ctp1
      - ctp2

  # Project external-secrets.io/v1beta1 SecretStores instances into namespaces matching the declared criteria.
  # If omitted, the secret store is projected to all namespaces within a control plane
  namespaceSelector:

    # matching any of the provided label sets
    matchLabels:
        team: team1

    # or by providing the list of namespace names
    names:
      - ns1
      - ns2
  
  # The rest of the fields is identical to .spec of ESO SecretStore
  # https://external-secrets.io/latest/api/secretstore/

  # You can specify retry settings for the http connection
  # these fields allow you to set a maxRetries before failure, and
  # an interval between the retries.
  # Current supported providers: AWS, Hashicorp Vault, IBM
  retrySettings:
    maxRetries: 5
    retryInterval: "10s"

  # provider field contains the configuration to access the provider
  # which contains the secret exactly one provider must be configured.
  provider:

    # (2) Hashicorp Vault
    vault:
      server: "https://vault.acme.org"

      # Path is the mount path of the Vault KV backend endpoint
      # Used as a path prefix for the external secret key
      path: "secret"

      # Version is the Vault KV secret engine version.
      # This can be either "v1" or "v2", defaults to "v2"
      version: "v2"

      # vault enterprise namespace: https://www.vaultproject.io/docs/enterprise/namespaces
      namespace: "a-team"

      # base64 encoded string of certificate
      caBundle: "..."

      # Instead of caBundle you can also specify a caProvider
      # this will retrieve the cert from a Secret or ConfigMap
      caProvider:
        # Can be Secret or ConfigMap
        type: "Secret"
        name: "my-cert-secret"
        key: "cert-key"

      auth:

        # static token: https://www.vaultproject.io/docs/auth/token
        tokenSecretRef:
          name: "my-secret"
          key: "vault-token"

        # AppRole auth: https://www.vaultproject.io/docs/auth/approle
        appRole:
          path: "approle"
          roleId: "db02de05-fa39-4855-059b-67221c5c2f63"
          secretRef:
            name: "my-secret"
            key: "vault-token"

        # Kubernetes auth: https://www.vaultproject.io/docs/auth/kubernetes
        kubernetes:
          mountPath: "kubernetes"
          role: "demo"

          # Optional service account reference
          serviceAccountRef:
            name: "my-sa"

          # Optional secret field containing a Kubernetes ServiceAccount JWT
          # used for authenticating with Vault
          secretRef:
            name: "my-secret"
            key: "vault"
```