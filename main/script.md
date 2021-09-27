- Deploy Kubewarden

  Rationale: straightforward to deploy.

  1. Add the helm repo
  1. Deploy the Kubewarden helm chart

    ```console
    $ helm repo add kubewarden https://charts.kubewarden.io
    "kubewarden" has been added to your repositories
    --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    $ helm install --wait --namespace kubewarden --create-namespace kubewarden-controller kubewarden/kubewarden-controller
    NAME: kubewarden-controller
    LAST DEPLOYED: Mon Sep 27 12:43:43 2021
    NAMESPACE: kubewarden
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    kubewarden-controller installed.

    You can start defining cluster admission policies by using the
    `clusteradmissionpolicies.policies.kubewarden.io` resource.

    For more information checkout https://kubewarden.io/
    ```

- Find a policy

  Rationale: we will now search for a policy that allows us to
  restrict privileged pods in our cluster. The Kubewarden Policy Hub
  is a place to find safe policies contributed by the Kubewarden
  project and others.

  1. Search in the Kubewarden Hub for "privileged"

- Deploy the pod-privileged-policy policy

  ```console
  $ kubectl apply -f - <<EOF
  apiVersion: policies.kubewarden.io/v1alpha2
  kind: ClusterAdmissionPolicy
  metadata:
    name: reject-privileged-pods
  spec:
    module: registry://ghcr.io/kubewarden/policies/pod-privileged:v0.1.9
    rules:
    - apiGroups: [""]
      apiVersions: ["v1"]
      resources: ["pods"]
      operations:
      - CREATE
      - UPDATE
    mutating: false
  EOF
  ```

  1. Wait for the policy to be active

    ```console
    $ kubectl wait --for=condition=PolicyActive clusteradmissionpolicy reject-privileged-pods
    clusteradmissionpolicy.policies.kubewarden.io/reject-privileged-pods condition met
    ```

  1. Try to create a pod with a privileged container
