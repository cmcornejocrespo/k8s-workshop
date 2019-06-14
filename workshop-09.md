# Workshop 09 - Helm the Kubernetes package manager


# Index
* [What is Helm?](#What-is-Helm?)
* [Using helm](#Using-helm)
  * [Exercise: Installing Helm](#Exercise:-Installing-Helm)
  * [Exercise: Helm Repos](#Exercise:-Helm-Repos)
  * [Exercise: Managing the life cycle of my first release](#Exercise:-Managing-the-life-cycle-of-my-first-release)

---

# What is Helm?

Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste

**Manage Complexity**

Charts describe even the most complex apps, provide repeatable application installation, and serve as a single point of authority.

**Easy Updates**

Take the pain out of updates with in-place upgrades and custom hooks.

**Simple Sharing**

Charts are easy to version, share, and host on public or private servers.

**Rollbacks**

Use `helm rollback` to roll back to an older version of a release with ease.

---


[Back to Index](#index)

---

# Using helm

**Main Helm Commands**

- Init: Set up Helm for the first time (`helm init`)
- Install: Install a chart (`helm install alpine`)
- Get, Status, List: Find out about charts (`helm list`)
- Repo Add, List, Remove, Index: Manage your helm repositories (`helm repo list`)
- Search: Search repos for charts (`helm search`)
- Create, Package: Create and package new charts (`helm create`)

## Exercise: Installing Helm

### Installing Helm client

There are various ways to install Helm Client: from binary, from source, using OS package managers or the installation script. You can find the installation details for your system [here](https://helm.sh/docs/using_helm/#installing-helm).


Check is installed

```sh
$ helm version

Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
```

### Installing Helm tiller

Tiller, the server portion of Helm, typically runs inside of your Kubernetes cluster. But for development, it can also be run locally, and configured to talk to a remote Kubernetes cluster.

**Note** RBAC is assumed to not be enabled.

**Command**
```sh
$ helm init
```

Check tiller is running

**Command**
```sh
$ kubectl get pods --show-labels --watch -n kube-system | grep tiller
```

---

[Back to Index](#index)

---

## Exercise: Helm Repos

A Helm repository is a collection of charts (packages) installable using Helm package manager and containing common configuration for Kubernetes apps such as resource definitions to configure the application’s runtime, various dependencies, communication mechanisms, and network settings.

By default, the Helm client interacts with the /stable  branch of the official Kubernetes Charts [repository](https://github.com/kubernetes/charts) that provides ~160 configured apps. The repository contains well-tested and well-maintained charts that comply with basic technical requirements of the Kubernetes. Redis, TensorFlow, NGINX, Apache Hadoop, Fluentd are just some examples of popular software available in this repository.

You can see the list of apps in the /stable repository by running the following command:

```sh
$ helm search
```


To find information about the individual chart, you can run `helm inspect <chart-name>`. For example, to learn more about the contents of stable/redis  chart, you can do the following:

**Command**
```sh
$ helm inspect stable/redis > redis.yaml
```

To find a full list of configurable options, you can use `helm inspect values <chart-name>` or find the official chart documentation in the chart repository on GitHub.

**Command**
```sh
$ helm inspect values stable/redis > redis-value.yaml
```

## Exercise: Managing the life cycle of my first release
**Objective** Learn how to fully manage the life cycle of a given existing chart.

You can customize the chart configuration during the installation. For example, to install the Redis chart with the server password and `persistence.enabled`  set to `false`.

```sh
$ helm install --name my-redis-release --set password=yoursecretpassword,persistence.enabled=false stable/redis
```

As you see, we are using --set  flag with a list of key/value pairs for the configuration options. Multiple values can be separated by ,  characters. So --set a=b,c=d  becomes:

```yml
a: b
c: d
```

We can check the current release we have deployed

**Command**

```sh
$ helm list
```

We can delete the release too:

**Command**

```sh
$ helm delete my-redis-release
$ helm list --deleted
```

We can delete the release permanently by running:

**Command**

```sh
$ helm delete --purge my-redis-release
```

Back to the previous example, if you are going to edit multiple configuration fields, it would be a good idea to save them in a file and refer to that file during the installation. For example, let’s create a file with the YAML-formatted configuration for our Redis chart:

```yml
password: mypassword
persistence:
  enabled: false
master:
  resources:
    Memory: 300Mi
    CPU: 200m
```

**Command**
```
$ sh vi redis-config.yml
```

First, delete the release running `helm delete my-redis-release --purge` and. Then, save the configuration above in the redis-config.yml  and install the Redis chart with the following command:

```sh
$ helm install --name my-redis-release -f redis-config.yml stable/redis
```

This will create the Redis release named `my-redis-release`. You can give any name to your release using the `--name`  flag on helm install

Check and discuss the output

As you see, Kubernetes has created a bunch of resources for your Redis app. If you are using `Minikube`, open the Kubernetes dashboard with minikube dashboard  to see them or use kubectl get commands to display them in your terminal.

**Command**

```sh
$ minikube dashboard
```

Alternatively you could run the following command ;-)

```sh
$ kubectl get all --selector=release=my-redis-release --show-labels
```

### Managing the life cycle of my first release

Helm client offers useful tools for managing your releases including upgrades and rollbacks. You may want to upgrade your release when a new version of the chart came out or when you need to change the chart’s configuration. To upgrade the release, you can use the `helm upgrade` command. This command will take configuration you provide and try to perform the least invasive upgrade updating only things that have changed since the last release.

To illustrate how the upgrade works, let’s first change our Redis configuration file `redis-config.yml`:

```yml
password: newpassword
persistence:
  enabled: false
master:
  resources:
    Memory: 300Mi
    CPU: 100m
  podLabels:
     mylabel: my-redis-v2
```

Now, let’s upgrade the release:

```sh
$ helm upgrade -f redis-config.yml my-redis-release stable/redis
```

Check new release history

```sh
$ helm history my-redis-release
```

Check new values

```sh
$ helm get values my-redis-release
```

Check new helm labels in pods

```sh
$ kubectl get pods --selector=release=my-redis-release --show-labels
```

You can rollback the release with `helm rollback [RELEASE] [REVISION]`. Now, let´s do a rollback.

```sh
$ helm rollback my-redis-release 1
```

Check release history

```sh
$ helm history my-redis-release
```

Check old helm labels in pods

```sh
$ kubectl get pods --selector=release=my-redis-release --show-labels
```

Check old values

```sh
$ helm get values my-redis-release
```

### Delete the Release

**Command**

```sh
$ helm del --purge my-redis-release
```

---

[Back to Index](#index)

---


## Exercise: Debugging Pending Pods
**Objectives:** Learn how to use `kubectl describe pod` and `kubectl describe nodes` to troubleshoot app deployments.

A common scenario that you can detect using events is when you’ve created a Pod that won’t fit on any node. For example, the Pod might request more resources than are free on any node, or it might specify a label selector that doesn’t match any nodes. Let’s say we created the previous Deployment with 5 replicas (instead of 2) and requesting 600 millicores instead of 500, on a four-node cluster where each (virtual) machine has 1 CPU. In that case one of the Pods will not be able to schedule

---

1) Start off by deleting the deployment and checking the node status

**Command**
```sh
$ kubectl delete deployment nginx-deployment
```

**Command**
```sh
$ kubectl describe nodes
```

Understand `kubectl describe nodes` **output**


3) Deploy and scale up previous deployment

**Command**
```
$ kubectl apply -f workshop-07/manifests/nginx-with-request.yaml
$ kubectl scale deploy nginx-deployment --replicas 5
```

4) Check pod status via `kubectl get pods`, as follows:

**Command**
```
$ kubectl get pods --show-labels --watch
```
Observe **pending** status

3) Find out what's happening using `kubectl describe`, as follows:

**Command**
```
$ kubectl describe pod nginx-deployment-<ANY_PENDING_POD_ID>
```

Here you can see the event generated by the scheduler saying that the Pod failed to schedule for reason FailedScheduling (and possibly others). The message tells us that there were not enough resources for the Pod on any of the nodes.

---

# Exercise: Installing Helm

There are various ways to install Helm Client: from binary, from source, using OS package managers or the installation script. You can find the installation details for your system here.

**Objectives:** Learn how to use the horizontal pod autoscaling feature and how to configure grafana to fetch pod/node metrics.

---

[Back to Index](#index)

---
---

# Helpful Resources

* [helm.sh](https://helm.sh/)
* [Helm Hub](https://hub.helm.sh/)
* [cluster-autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)
* [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)

---

[Back to Index](#index)

---