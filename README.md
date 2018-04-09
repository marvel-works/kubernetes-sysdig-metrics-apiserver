# k8s-sysdig-adapter

Table of contents:

- [Introduction](#introduction)
- [Installation](#installation)
- [Playground](#playground)
- [Links](#links)

## Introduction

`k8s-sysdig-adapter` is an implementation of the [Custom Metrics API
(`custom.metrics.k8s.io`)][custom-metrics-api-types] using Sysdig Monitor.

Essentially, this component is a custom Kubernetes API server that queries
Sysdig Monitor's API for metrics data and exposes it to Kubernetes. You can
think of it as a channel adapter between Sysdig and the
[Horizontal Pod Autoscaling API][hpa] for Kubernetes.

Once running you should be able to feed your a `HorizontalPodAutoscaler` objects
(autoscalers) in the `autoscaling/v2beta1` form like the one below:


```yaml
kind: HorizontalPodAutoscaler
apiVersion: autoscaling/v2beta1
metadata:
  name: kuard
spec:
  scaleTargetRef:
    kind: Deployment
    name: kuard
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Object
    object:
      target:
        kind: Service
        name: kuard
      metricName: net.http.request.count
      targetValue: 100
```

This autoscaler is based on the `net.http.request.count` metric: 100 reqs/min.
The autoscaler will adjust the number of pods deployed as the metric fluctuates
over or below the target.

## Installation

:construction:

## Playground

The playground is designed to run `k8s-sysdig-metrics` locally using virtual
machines. We provision a Kubernetes cluster with a single controller node and a
configurable number of worker nodes.

This playground borrows some ideas from other Vagrant-based environments I've
found in projects like [kubevirt][p1], [k8s-snowflake][p2] or
[kubernetes-ansible-vagrant][p3]. Thank you!

### Requirements

**Please use the most recent versions of Vagrant and VirtualBox!**

The controller node is assigned 1GB of RAM. By default we're creating two
worker nodes - each is assigned 2GB of RAM.

### Installation

To create the environment run:

    ./playground/setup.sh

Optionally, you can define the number of worker nodes and/or enable the debug
mode so the shell scripts become verbose, e.g.:

    env DEBUG=1 WORKERS=2 ./playground/setup.sh

This script is going to do a few things for us:

- It runs `vagrant up` for us to provision the virtual machines, which uses
[bootstrap.sh][p4] to install the necessary packages and apply a few tweaks on
each node so the Kubernetes components can run properly.

- It runs [kubeadm.sh][p5] which uses [kubeadm][p6] to set up the Kubernetes
cluster. The kubeconfig generated is copied in a temporary directory of the host
machine and its location is printed before the script ends so you can use it to
operate the cluster.

If the script completes successfully you should see something like the
following:

```
We're done, thank you for waiting!
kubectl config available in /tmp/tmp.8nJG5UCHA7/config

Usage example:
$   export KUBECONFIG=/tmp/tmp.8nJG5UCHA7/config
$   kubectl get nodes
-------------------------------------------------------------------------
```

Copy the `config` file somewhere else if your temporary folder is not
persistent. You can optionally consolidate the output with your local
`.kube/config` - we thought it would be safer if you do that manually!

### Check the status of the cluster

Let's make sure that the cluster is running as expected. We're going to export
a custom `KUBECONFIG` environment string so we don't have to pass the location
on every command.

    $ export KUBECONFIG=/tmp/tmp.8nJG5UCHA7/config

Let's confirm that all the nodes are listed as ready:

    $ kubectl get nodes
    NAME              STATUS    ROLES     AGE       VERSION
    controller-node   Ready     master    4m        v1.10.0
    worker-node-1     Ready     <none>    4m        v1.10.0
    worker-node-2     Ready     <none>    4m        v1.10.0

Now let's confirm that the core components are in a healthy state:

    $ kubectl get componentstatuses
    NAME                 STATUS    MESSAGE              ERROR
    controller-manager   Healthy   ok
    scheduler            Healthy   ok
    etcd-0               Healthy   {"health": "true"}

You're ready! :tada:

## Links

From the Kubernetes project:

- [Types in Custom Metrics API][l1]
- [Library for writing a Custom Metrics API server][l2]
- [Library for writing a Kubernetes-style API Server][l3]
- Documentation: [Horizontal Pod Autoscaling][l4]

Other links:

- [Sysdig's blog post about HPA][l5]
- [Kubernetes Prometheus Adapter][l6] 

[custom-metrics-api-types]: https://github.com/kubernetes/metrics/tree/master/pkg/apis/custom_metrics
[hpa]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#horizontalpodautoscaler-autoscaling-v2beta1-
[p1]: https://github.com/kubevirt/kubevirt
[p2]: https://github.com/jessfraz/k8s-snowflake
[p3]: https://github.com/errordeveloper/kubernetes-ansible-vagrant
[p4]: ./bootstrap.sh
[p5]: ./kubeadm.sh
[l1]: https://github.com/kubernetes/metrics/tree/master/pkg/apis/custom_metrics
[l2]: https://github.com/kubernetes-incubator/custom-metrics-apiserver
[l3]: https://github.com/kubernetes/apiserver
[l4]: https://github.com/kubernetes/community/blob/master/contributors/design-proposals/autoscaling/horizontal-pod-autoscaler.md
[l5]: https://sysdig.com/blog/kubernetes-scaler/
[l6]: https://github.com/directXMan12/k8s-prometheus-adapter/
