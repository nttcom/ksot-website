---
title: "Concepts"
linkTitle: "Concepts"
weight: 3
description: >
  Learn more about the core concepts of Kuesta.
---


## Network Configuration as Code with CUE

Kuesta supports creating an abstracted high-level configuration layer to expose easy high-level data model and API.
The configuration data models of the network devices have lots of networking capabilities therefore are too much detailed and complicated, they are not suitable for the resource model of the Infrastructure as Code.
To provide a data model which can represent the user intent easily, an abstracted high-level data model is required to hide the complicated actual device configurations.
In addition, the high-level data model which effects multiple network device configs is needed for domain-wide abstraction such as E2E connectivity.

These high-level data model abstraction causes the additional complexity of the high-to-low data mapping logic.
The relations between the high-level model and the low-level actual device configs are many-to-many, we need to consider performing data mapping and data composition without data loss, conflicts, and type constraint violation.
A simple text-templating approach using Python/Jinja is not enough to solve this complex problem.

Kuesta uses CUE as a programming language of high-to-low data mapping logic. CUE is a configuration language specialized in data unification and validation, and is well-suited for the above usecase:
- CUE enables us to unify multiple document-tree in the arbitrary layer. CUE is designed to ensure combining CUE values in any order always gives the same result (associative, commutative, and idempotent).
- CUE merges types and values into a single concept. Even types and constraints are the kind of value, the only difference between them is that they do not have concrete value. Due to this novel approach, we can declare constraints and schema simply and efficiently in the configuration data itself.
- Cue is highly programmable and supports software coding practices like templating and modularization. We can take the same advantages as if we use general-purpose language.

To learn more about CUE, see ["cuelang.org"](https://cuelang.org/docs/about/).


## GitOps

Kuesta provides GitOps for network configuration. It aims to describe the entire network configuration declaratively and version controlled in a Git repository, as a Single Source of Truth(SSoT). 
The configuration process is totally automated so that the deployed network configuration matches the state specified in a repository. All you have to do to switch the network configuration or rollback is to change the Git branch HEAD to the revision you want to deploy.

Kuesta selects Pull-based approach for the following reasons:
- To detect the configuration drift between the desired config in the configuration repository and the actual config stored in devices, whenever the device config is updated.
- To ensure that the entire network configuration stored at the SSoT git repository is deployed to the devices with the prepared deploy pipeline, without inserting any configuration changes.
- To avoid aggregating all device credentials and providing them to the single god-mode configuration system. Device credentials are stored as the Kubernetes Secret resource, and you can integrate with public clouds' SecretManager to manage these credentials more securely using OSS tools like ExternalSecrets Operator.

For more information about GitOps, take a look at [“What is GitOps?”](https://www.gitops.tech/#what-is-gitops).


## Kubernetes Custom Operator

Kuesta uses the Kubernetes reconciliation loop and the operator pattern as an engine of Infrastructure as Code. 
Kubernetes is a well-known container orchestration tool, but it can be extended to automate deploying any external resources including network devices.
Kuesta consists of multiple Kubernetes custom operators to perform GitOps, distributed transaction coordinator, and the driver to configure the network devices.

You can extend Kuesta to support new vendors' devices or new versions by implementing Kubernetes custom operator for them as a device driver.


## The components of Kuesta

Kuesta consists of the following components:

- **kuesta-server** is the core api-server of Kuesta. It exposes gNMI to integrate with the external 3rd-party system, performs high-to-low data mapping and data composition, and creates new git-commit and PullRequest for GitOps.

- **kuesta-aggregator** aggregates the actual device config changes and creates a new git-commit to persist the config change history.

- **FluxCD source-controller** detects manifest changes of the specified Git repository at `GitRepository` Kubernetes custom resource.

- **kuesta-provisioner** consists of the gitrepository-watcher controller and `DeviceRollout` Kubernetes custom resource. `gitrepository-watcher` watches `GitRepository` status to detect the manifest changes and updates `DeviceRollout` status  to `running` when change is detected, which triggers the device config update transaction.

To configure network devices using Kuesta, you must prepare your own Kubernetes custom operator to configure your target devices.
It is recommended to use [kubebuilder](https://github.com/kubernetes-sigs/kubebuilder) which is the powerful framework to build your Kubernetes custom operator.
You can also start from [device-operator](https://github.com/nttcom/kuesta/tree/main/device-operator) used at the [getting-started](/docs/getting-started), the sample Kubernetes custom operator designed to configure OpenConfig/gNMI device.
It is built using kubebuilder.
