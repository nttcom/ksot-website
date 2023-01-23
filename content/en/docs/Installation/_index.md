---
title: "Installation"
linkTitle: "Installation"
weight: 2
description: >
  Instructions to install Kuesta components.
---

Information in this section helps you to install Kuesta to your Kubernetes cluster.

## Before you begin

You must have a Kubernetes cluster running version 1.24 or later.

If you don’t have a cluster, you can create local one for testing with kind.
[Install kind](https://kind.sigs.k8s.io/docs/user/quick-start/) and create a cluster by running `kind create cluster`. 
This will create a cluster running locally, with RBAC enabled and your user granted the cluster-admin role.


## Prerequisites

1. [Install kubectl](https://kubernetes.io/docs/tasks/tools/) to run commands against Kubernetes clusters.
2. [Install cue](https://cuelang.org/docs/install/) to run Kuesta installation script.


## Installing Kuesta on Kubernetes

To install Kuesta components, just run cue script:

```bash
git clone https://github.com/nttcom/kuesta.git
cd kuesta/tool/install
cue install
```

If you want to install a specific release version or install from the other container image registry, use `version` and `imageRegistry` tag:

```bash
cue -t imageRegistry=<your_image_registry> -t version=<your_version> install
```

Provide required parameters according to the instructions:

```bash
GitHub repository for config: https://github.com/<your_org>/<your_config_repo>
GitHub repository for status: https://github.com/<your_org>/<your_status_repo>
GitHub username: <your_username>
GitHub private access token: <your_private_access_token>
Are these git repositories private? (yes|no): yes
Do you need sample driver and emulator for trial?: (yes|no) yes

---
GitHub Config Repository: https://github.com/<your_org>/<your_config_repo>
GitHub Status Repository: https://github.com/<your_org>/<your_status_repo>
GitHub Username: <your_username>
GitHub Access Token: ***
Use Private Repo: true

Image Registry: ghcr.io/nttcom/kuesta
Version: latest
Deploy sample driver and emulator: true
---

Continue? (yes|no) yes
```

For Kuesta to perform git-push and create pull-request, you need to prepare a GitHub [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) (PAT) and provide it in the installation.
PAT provided here is stored in a Secret named `kuesta-system/kuesta-secrets` for kuesta-server to operator git clone, pull, push and create PullRequest.

You can also use this PAT when you set up `GitRepository` to watch your private repository via Git over HTTPS, to grant Flux source-controller to conduct git operations with your private repository.


## Configuring GitOps with Flux source-controller

To set up GitOps using Flux source-controller, you need to create `GitRepository` to define the IaC manifest source.
The following is the example of a `GitRepository`, that is created in the [getting-started](/docs/getting-started) tutorial.

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: kuesta-testdata
  namespace: kuesta-getting-started
spec:
  interval: 1m0s
  url: https://github.com/hrk091/kuesta-testdata
  ref:
    branch: main
  secretRef:
    name: kuesta-testdata-secret
  timeout: 60s
```

You can get more detail from [the FluxCD documentation](https://fluxcd.io/flux/components/source/gitrepositories/).

In Kuesta, once you create the `GitRepository`, Kuesta's source-watcher controller detects it and creates the corresponding `DeviceRollout`.
`DeviceRollout` is a custom resource responsible for multi-device transaction management. The `DeviceRollout` created in the getting-started tutorial is as follows:

```
apiVersion: kuesta.hrk091.dev/v1alpha1
kind: DeviceRollout
metadata:
  name: kuesta-testdata
  namespace: kuesta-getting-started
spec:
  deviceConfigMap:
    oc01:
      checksum: 1524caf17ec4c133f6f275326a6c9742a0155007f8eaa28ea601ce7ac39c0566
      gitRevision: main/30746f9beb48d237b6c0e0357708b3ed1f95cd1b
    oc02:
      checksum: 0b03381195ba613e0dd20e5d2683711e9d6082f0634bee7c54014de2eadf0fc9
      gitRevision: main/30746f9beb48d237b6c0e0357708b3ed1f95cd1b
status:
  desiredDeviceConfigMap:
    oc01:
      checksum: 1524caf17ec4c133f6f275326a6c9742a0155007f8eaa28ea601ce7ac39c0566
      gitRevision: main/30746f9beb48d237b6c0e0357708b3ed1f95cd1b
    oc02:
      checksum: 0b03381195ba613e0dd20e5d2683711e9d6082f0634bee7c54014de2eadf0fc9
      gitRevision: main/30746f9beb48d237b6c0e0357708b3ed1f95cd1b
  deviceStatusMap:
    oc01: Completed
    oc02: Completed
  phase: Healthy
  status: Completed      
```

The lifecycle of `DeviceRollout` resource is completely automated, you don't need to touch it at all. 


## Installing Device Driver and configuring Device Custom Resource

{{% pageinfo %}}

The following instructions are just for the sample Kubernetes Custom Operator that is used in the getting-started tutorial.
The details will be different depending on the target devices you want to configure by Kuesta and your Kubernetes custom operator to configure them.

{{% /pageinfo %}}

To configure the devices which supports OpenConfig over gNMI, you can use the sample Kubernetes custom operator developed at [kuesta/device-operator](https://github.com/nttcom/kuesta/tree/main/device-operator). This sample Kubernetes custom operator is specialized
to configure OpenConfig devices which support [basic YANG sets](https://github.com/nttcom/kuesta/tree/main/device-operator/pkg/model/yang),
you can extend this custom operator by adding any YANG modules which is written in [OpenConfig style](https://github.com/openconfig/public/blob/master/doc/openconfig_style_guide.md).


You can install this sample Kubernetes custom operator either using `make` command or using `kubectl/kustomize` directly,
but the fastest way is using the Kuesta installation script. It uses `kustomize` internally by creating kustomize overlay directory named `getting-started`, you can start with it to customize your setup.

To install the sample Kubernetes custom operator using `getting-started` overlay directory, run the following:

```bash
export IMG=ghcr.io/nttcom/kuesta/device-operator:latest
export KUSTOMIZE_ROOT=overlays/getting-started
make deploy
```

To configure devices from Kuesta, you also need to create custom resources for each device in addition to the Kubernetes custom operator.
In the getting-started tutorial, following `OcDemo` custom resource is created for oc01 gNMI emulator.

```yaml
apiVersion: kuesta.hrk091.dev/v1alpha1
kind: OcDemo
metadata:
  name: oc01
  namespace: kuesta-getting-started
spec:
  address: gnmi-fake-oc01.kuesta-getting-started
  port: 9339
  rolloutRef: kuesta-testdata
  tls:
    secretName: oc01-cert
    skipVerify: true
```

`OcDemo` defines a connection parameters to access the configuration target device, address, port, username/password, and TLS settings.
`.spec.rolloutRef` is also a required field to specify `DeviceRollout` to subscribe to the device configuration changes given by Flux source-controller GitOps. 
For more information, run `kubectl explain ocdemo.spec`.
