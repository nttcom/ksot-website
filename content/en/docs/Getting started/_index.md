---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 1
description: >
  Get started with Kuesta.
---

This tutorial shows you how to

1. Create a local Kubernetes cluster.
2. Install Kuesta.
3. Run device emulator.
4. Create new service and configure the device emulators by GitOps.


## Prerequisites

1. [Install golang (latest)](https://go.dev/doc/install) to use controller-tools.
2. [Install docker (latest)](https://docs.docker.com/engine/install) to use cluster in kind.
3. [Install kind (latest)](https://kind.sigs.k8s.io/docs/user/quick-start/) to set up local Kubernetes cluster.
4. [Install kubectl (latest)](https://kubernetes.io/docs/tasks/tools/) to run commands against Kubernetes clusters.
5. [Install cue (v0.4.x)](https://cuelang.org/docs/install/) to run Kuesta installation script.
6. [Install gnmic (latest)](https://gnmic.kmrd.dev/install/) to perform gNMI request to kuesta server.


## Create a local Kubernetes cluster

Create a local Kubernetes cluster with kind.

```bash
kind create cluster
```

You can check that the cluster was successfully created by running:

```bash
kubectl cluster-info
```

Then you will see an output similar to the following that confirms your local cluster is running:

```
Kubernetes control plane is running at https://127.0.0.1:52096
CoreDNS is running at https://127.0.0.1:52096/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## Set up GitHub source repositories for testing

To configure network devices using Kuesta's GitOps, you need to set up 2 GitHub repositories. One is for
the configuration source repository, and the other is for the update history of network devices' actual configuration status.
You can easily set up these repositories using [Kuesta Example](https://github.com/nttcom/kuesta-example) repository as a template.
Repository must consist of lower case alphanumeric characters, '-' or '.', and must start and end with an alphanumeric character.

Run the following 2 times to create the repository pair:

1. Click **Use this template** button of [Kuesta Example repository](https://github.com/nttcom/kuesta-example).

2. Select a name for your new repository and click **Create repository from template**. You can choose either public or private whichever you prefer.


## Install Kuesta and sample resources

To install Kuesta and sample resources like device drivers and emulators, just run cue script:

```bash
git clone https://github.com/nttcom/kuesta.git
cd kuesta/tool/install
cue install
```

Provide required parameters according to the instructions:
The `GitHub repository for config` `GitHub repository for status` should specify each of the two Github repositories for verification that you have created.

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

For Kuesta to perform git-push and create pull-request, you need to prepare a GitHub [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) (PAT) and provide it in the installation.(classic is recommended)
Since PAT provided here is stored only in Secret resources on your local Kubernetes cluster, you can remove them safely and completely by [cleaning up your local cluster](/docs/getting-started/#clean-up).

After running an installation script, you can see what is installed by running `kubectl` commands:

1: Custom Resource Definitions(CRDs):
```bash
kubectl get customresourcedefinitions.apiextensions.k8s.io
````

```bash
NAME                                        CREATED AT
buckets.source.toolkit.fluxcd.io            2023-01-10T06:31:20Z
certificaterequests.cert-manager.io         2023-01-10T06:31:19Z
certificates.cert-manager.io                2023-01-10T06:31:19Z
challenges.acme.cert-manager.io             2023-01-10T06:31:19Z
clusterissuers.cert-manager.io              2023-01-10T06:31:19Z
devicerollouts.kuesta.hrk091.dev            2023-01-10T06:43:30Z
gitrepositories.source.toolkit.fluxcd.io    2023-01-10T06:31:20Z
helmcharts.source.toolkit.fluxcd.io         2023-01-10T06:31:20Z
helmrepositories.source.toolkit.fluxcd.io   2023-01-10T06:31:20Z
issuers.cert-manager.io                     2023-01-10T06:31:19Z
ocdemoes.kuesta.hrk091.dev                  2023-01-10T06:43:36Z
ocirepositories.source.toolkit.fluxcd.io    2023-01-10T06:31:20Z
orders.acme.cert-manager.io                 2023-01-10T06:31:19Z
```

Some CRDs belonging to `kuesta.hrk091.dev`, `source.toolkit.fluxcd.io`, `cert-manager.io` are installed.


2: Pods:
```bash
kubectl get pods -A | grep -v 'kube-system'
```

```bash
NAMESPACE                NAME                                                  READY   STATUS    RESTARTS   AGE
cert-manager             cert-manager-7475574-t7qlm                            1/1     Running   0          12m
cert-manager             cert-manager-cainjector-d5dc6cd7f-wbh6r               1/1     Running   0          12m
cert-manager             cert-manager-webhook-6868bd8b7-kc85s                  1/1     Running   0          12m
device-operator-system   device-operator-controller-manager-5d8648469b-57x4l   2/2     Running   0          11s
flux-system              source-controller-7ff779586b-wsgf7                    1/1     Running   0          12m
kuesta-getting-started   gnmi-fake-oc01-79d4d679b4-tvm78                       1/1     Running   0          10s
kuesta-getting-started   gnmi-fake-oc02-69d9cc664d-56bsz                       1/1     Running   0          10s
kuesta-getting-started   subscriber-oc01                                       1/1     Running   0          9s
kuesta-getting-started   subscriber-oc02                                       1/1     Running   0          9s
kuesta-system            kuesta-aggregator-7586999c47-jj7m4                    1/1     Running   0          27s
kuesta-system            kuesta-server-85fc4f8646-kq72c                        1/1     Running   0          27s
local-path-storage       local-path-provisioner-9cd9bd544-lk9w9                1/1     Running   0          16m
provisioner-system       provisioner-controller-manager-7675d487c8-w6f7x       1/2     Running   0          17s
```

When all pods show Running status, you are ready to continue.

Kuesta core services are deployed in the following namespaces:

- kuesta-system
  - Kuesta Server which provides core features and serves API.
- provisioner-system
  - Kubernetes custom operator used for GitOps and multi-device transaction management.
- flux-system
  - Flux source-controller and Custom Resources to integrate with manifest sources.
- cert-manager
  - Act as a private CA and issue certificates for mTLS.

The resources for this getting-started are deployed as well in the following namespaces:

- device-operator-system
  - Kubernetes custom operator as a device driver to configure device emulator.
- kuesta-getting-started
  - Device emulators for this getting started, and Kubernetes custom resources to trigger device configuration.

Following 2 pods are device emulators which can be configured by OpenConfig over gNMI, named `oc01` and `oc02`.

```bash
kuesta-getting-started   gnmi-fake-oc01-79d4d679b4-tvm78                       1/1     Running   0          10s
kuesta-getting-started   gnmi-fake-oc02-69d9cc664d-56bsz                       1/1     Running   0          10s
```

If you want to check what configurations are stored in these emulators, you can easily do it
by port-forwarding and requesting to these emulator pods as follows:

```bash
# Get oc01 emulator's configuration
kubectl -n kuesta-getting-started port-forward svc/gnmi-fake-oc01 9339:9339
gnmic -a :9339 -u admin -p admin get --path "" --encoding JSON --skip-verify
```


## Request Service configuration and run GitOps

A **Service** is a abstracted high-level model of the network configuration, and you can configure multiple devices
by creating/updating Services.

In the sample repository there is a sample Service named `oc-circuit`, you can try it as a demonstration.
Service is written by CUE. For example, `oc-circuit` service is as follows:

```cue
package oc_circuit

import (
	ocdemo "github.com/hrk091/kuesta-testdata/pkg/ocdemo"
)

#Input: {
	// kuesta:"key=1"
	vlanID: uint32
	endpoints: [...{
		device: string
		port:   uint32
	}]
}

#Template: {
	input: #Input

	output: devices: {
		for _, ep in input.endpoints {
			"\(ep.device)": config: {
				ocdemo.#Device

				let _portName = "Ethernet\(ep.port)"
				Interface: "\(_portName)": SubInterface: "\(input.vlanID)": {
					Ifindex:     ep.port
					Index:       input.vlanID
					Name:        "\(_portName).\(input.vlanID)"
					AdminStatus: 1
					OperStatus:  1
				}

				Vlan: "\(input.vlanID)": {
					VlanId: input.vlanID
					Name:   "VLAN\(input.vlanID)"
					Status: ocdemo.#Vlan_Status_ACTIVE
					Tpid:   ocdemo.#OpenconfigVlanTypes_TPID_TYPES_UNSET
				}
			}
		}
	}
}
```

The `oc-circuit` Service will conduct mapping from `#Template.input` (Service config) to the `#Template.output` (multiple network device configs) as defined in `#Template`, then it will generate simple vlan and vlan sub-interface configurations.
To use `oc-circuit` Service, you need to provide vlanID and multiple device ports according to the `#Input` type definition.

You can create the `oc-circuit` Service by the following steps:

1: Create a request json file named `oc-circuit-vlan100.json` with the following content:

```json
{
    "vlanID": 100,
    "endpoints": [
        {
            "device": "oc01",
            "port": 1
        },
        {
            "device": "oc02",
            "port": 1
        }
    ]
}
```

This sample Service requests both oc01 and oc02 device emulators to create a VLAN sub-interface with vlanID=100 on port 1.

2: Port-forward to the kuesta-server Pod running on your local Kubernetes.

```bash
kubectl -n kuesta-system port-forward svc/kuesta-server 9339:9339
```

3: Send gNMI set request to kuesta-server using `gnmic`.

```bash
gnmic -a :9339 -u dummy -p dummy set --replace-path "/services/service[kind=oc_circuit][vlanID=100]" --encoding JSON --insecure --replace-file oc-circuit-vlan100.json
```

4: Check the PullRequest(PR) in your configuration repository on GitHub web console. Access [PR list](https://github.com/<your_org>/<your_config_repo>/pulls) then you will find the PR which titles as `[kuesta] Automated PR`. You can see what services and devices are configured by this PR on the PR comment, and their details from the `Files changed` tab.

5: Before merging PR branch, it is better to monitor `DeviceRollout` resource, which conducts a device configuration rollout. Run monitor with `kubectl`:

```bash
kubectl -n kuesta-getting-started get devicerollouts.kuesta.hrk091.dev -w
```

If you see there is one `DeviceRollout` resource and its `STATUS` is `Completed`, you are ready to run GitOps delivery procedure.

```bash
NAME              PHASE     STATUS
kuesta-testdata   Healthy   Completed
```


6: Merge the PR on GitHub web console, and go to the terminal where `DeviceRollout` monitor running.
In one minute, you will see `DeviceRollout` status changes to `Running`, then changes back to `Completed` again.

```bash
NAME              PHASE     STATUS
kuesta-testdata   Healthy   Completed
kuesta-testdata   Healthy   Completed
kuesta-testdata   Healthy   Running
kuesta-testdata   Healthy   Running
kuesta-testdata   Healthy   Running
kuesta-testdata   Healthy   Completed
```

7: Check the updated device configuration. Get entire device config by sending gNMI get request to kuesta-server with `gnmic`:

```bash
gnmic -a :9339 -u admin -p admin get --path "/devices/device[name=oc01]" --path "/devices/device[name=oc02]" --encoding JSON --insecure
```

The output displays the gNMI response payload like:

```json
[
  {
    "source": ":9339",
    "timestamp": 1673360105000000000,
    "time": "2023-01-10T23:15:05+09:00",
    "updates": [
      {
        "Path": "devices/device[name=oc01]",
        "values": {
          "devices/device": {
            "Interface": {
              "Ethernet1": {
                "AdminStatus": 1,
                "Description": "awesome interface!!",
                "Enabled": false,
                "Mtu": 9000,
                "Name": "Ethernet1",
                "OperStatus": 1,
                "Subinterface": {
                  "100": {
                    "AdminStatus": 1,
                    "Ifindex": 1,
                    "Index": 100,
                    "Name": "Ethernet1.100",
                    "OperStatus": 1
                  },
                  ...
                },
                ...
              },
              ...
            },
            "Vlan": {
              "100": {
                "Member": [],
                "Name": "VLAN100",
                "Status": 1,
                "Tpid": 0,
                "VlanId": 100
              },
              ...
```

You will find that vlan and vlan sub-interfaces of both devices oc01 and oc02 are configured correctly as implemented in the `oc-circuit` service model.
In addition, these device config changes are committed and pushed to your status repository, you can find these device config updates from the main branch head.


## Clean up

To delete the local cluster you created for this getting-started, just run:

```bash
kind delete cluster
```

Do not forget to remove your PAT to access your private repository if no longer needed.
