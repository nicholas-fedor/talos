---
title: Docker
description: "Creating Talos Kubernetes cluster using Docker."
aliases:
  - ../../../local-platforms/docker
---

In this guide we will create a Kubernetes cluster in Docker, using a containerized version of Talos.

Running Talos in Docker is intended to be used for CI pipelines and local testing when you need a quick and easy cluster.
If you are running Talos in production, Docker provides an excellent way for developers to develop against the same version of Talos.

## Requirements

The follow are requirements for running Talos in Docker:

- Docker 18.03 or greater
- A recent version of [`talosctl`](https://github.com/siderolabs/talos/releases)

## Caveats

Certain APIs are not available when Talos is running in a container. For example, `upgrade`, `reset`, and similar APIs don't apply in container mode.

VIPs are not supported when running on a Mac in Docker due to networking limitations.

## Create the Cluster

Bootstrap a local cluster:

```bash
talosctl cluster create
```

- > Startup times can take a few minutes before the cluster is available.

- The Talos and Kubernetes API's are mapped to a random port on the host machine.
- Once the process finishes successfully, `talosconfig` (`~/.talos/config`) and
  `kubeconfig` (`~/.kube/config`) will be configured to point to the new cluster
  `talos-default`.

Specify which nodes you want to communicate with:

```bash
talosctl config nodes 10.5.0.2 10.5.0.3
```

- Talosctl can operate on one or all the nodes in the cluster – this makes
  cluster wide commands much easier.

View the Talos API endpoint:

```bash
talosctcl config info
...
Endpoints:           127.0.0.1:38423
```

View the Kubernetes API endpoint:

```bash
talosctl cluster show
...
KUBERNETES ENDPOINT   https://127.0.0.1:43083
```

## Using the Cluster

Once the cluster is available, you can make use of `talosctl` and `kubectl` to
interact with the cluster.

To view currently running containers:

- For the `system` namespace:

  ```bash
  talosctl containers
  ```

- For the `k8s.io` namespace:

  ```bash
  talosctl containers -k
  ```

To view the logs of a container:

- For the `system` namespace:

  ```bash
  talosctl logs <container>
  ```

- For the `k8s.io` namespace:

  ```bash
  talosctl logs -k <container>
  ```

## Cleaning Up

To cleanup, run:

```bash
talosctl cluster destroy
```

## Multiple Clusters

Multiple Talos Linux cluster can be created on the same host, each cluster will need to have:

- a unique name (default is `talos-default`)
- a unique network CIDR (default is `10.5.0.0/24`)

To create a new cluster:

- `talosctl` will set `cluster2` as the current context.

```bash
talosctl cluster create --name cluster2 --cidr 10.6.0.0/24
```

To destroy a specific cluster:

```bash
talosctl cluster destroy --name cluster2
```

To switch between clusters, use `--context` flag:

```bash
talosctl --context cluster2 version
kubectl --context admin@cluster2 get nodes
```

## Running Talos in Docker Manually

To run Talos in a container manually:

```bash
docker run --rm -it \
  --name tutorial \
  --hostname talos-cp \
  --read-only \
  --privileged \
  --security-opt seccomp=unconfined \
  --mount type=tmpfs,destination=/run \
  --mount type=tmpfs,destination=/system \
  --mount type=tmpfs,destination=/tmp \
  --mount type=volume,destination=/system/state \
  --mount type=volume,destination=/var \
  --mount type=volume,destination=/etc/cni \
  --mount type=volume,destination=/etc/kubernetes \
  --mount type=volume,destination=/usr/libexec/kubernetes \
  --mount type=volume,destination=/opt \
  -e PLATFORM=container \
  ghcr.io/siderolabs/talos:{{< release >}}
```

The machine configuration submitted to the container should have a [host DNS feature]({{< relref "../../../reference/configuration/v1alpha1/config#Config.machine.features.hostDNS"  >}}) enabled with `forwardKubeDNSToHost` enabled.
It is used to forward DNS requests to the resolver provided by Docker (or other container runtime).
