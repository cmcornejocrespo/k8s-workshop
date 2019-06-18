# Workshop 07 - Monitoring

Application and systems logs can help you understand what is happening inside your cluster. The logs are particularly useful for debugging problems and monitoring cluster activity.

Kubernetes provides no native storage solution for log data, but you can integrate many existing logging solutions into your Kubernetes cluster.

This tutorial covers the fundamental building blocks that make up Kubernetes. Understanding what these components are
and how they are used is crucial to learning how to use the higher level objects and resources.

# Index
* [Application Introspection and Debugging](#Application-Introspection-and-Debugging)
  * [Exercise: kubectl top pod](#Exercise:-kubectl-top-pod)
  * [Exercise: kubectl describe pods](#Exercise:-kubectl-describe-pods)
  * [Exercise: Debugging Pending Pods](#Exercise:-debugging-pending-pods)
* [Cluster-level monitoring](#Cluster-level-monitoring)
  * [Exercise: kubectl top node](#Exercise:-kubectl-top-node)
  * [Exercise: Kubernetes dashboard](#Exercise:-kubernetes-dashboard)
  * [Exercise: Deploy weave scope](#Exercise:-deploy-weave-scope)
  * [Exercise: Deploy stack elastic](#Exercise:-deploy-stack-elastic)
* [Cleaning up](#cleaning-up)
* [Helpful Resources](#helpful-resources)

---

# Application Introspection and Debugging
Once your application is running, you’ll inevitably need to debug problems with it. You should be able to troubleshoot your application at pod and node level by leveraging to the Kubernetes API commands.

---

## Exercise: kubectl top pod
**Objectives:** Learn how to use `kubectl top pod` to fetch metrics about pods.

The easiest way to see basic node metrics is by using kubectl. The kubectl top command communicates with Metrics Server to get the latest node metrics and displays them in your terminal.

1) Deploy example pod

        **Command**
        ```sh
        $ kubectl apply -f workshop-07/manifests/nginx-with-request.yaml
        ```

2) Check pod metrics (it may take time)

    **Command**

    ```
    $ kubectl top pod

    NAME                                CPU(cores)   MEMORY(bytes)   
    nginx-deployment-86b474f84f-bgrqd   0m           2Mi             
    nginx-deployment-86b474f84f-gv66s   0m           2Mi  
    ```

3) Clean up

    **Command**

    ```
    $ kubectl delete deployment nginx-deployment
    ```

---

[Back to Index](#index)

---

## Exercise: kubectl describe pods
**Objectives:** Learn how to use `kubectl describe pod` to fetch details about pods.

For this example we’ll use a Deployment to create two pods initially.

---

1) Deploy example pod

    **Command**
    ```
    $ kubectl apply -f workshop-07/manifests/nginx-with-request.yaml
    ```

2) Check pod status

    **Command**

    ```
    $ kubectl get pods --watch
    ```

3) We can retrieve a lot more information about each of these pods using kubectl describe pod:

    **Command**
    ```
    $ kubectl describe pod $(kubectl get pods --selector=app=nginx -o jsonpath='{.items[0].metadata.name}')
    ```

    Here you can see configuration information about the container(s) and Pod (labels, resource requirements, etc.), as well as status information about the container(s) and Pod (state, readiness, restart count, events, etc.).

    - The container **state** is one of Waiting, Running, or Terminated. Depending on the state, additional information will be provided – here you can see that for a container in Running state, the system tells you when the container started.

    - **Ready** tells you whether the container passed its last readiness probe. (In this case, the container does not have a readiness probe configured; the container is assumed to be ready if no readiness probe is configured.)

    - **Restart** Count tells you how many times the container has been restarted; this information can be useful for detecting crash loops in containers that are configured with a restart policy of ‘always.’

    - Currently the only **Condition** associated with a Pod is the binary Ready condition, which indicates that the pod is able to service requests and should be added to the load balancing pools of all matching services.

    - Lastly, you see a log of recent **events** related to your Pod

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


2) Deploy and scale up previous deployment
    **Command**
    
    ```sh
    $ kubectl apply -f workshop-07/manifests/nginx-with-request.yaml
    $ kubectl scale deploy nginx-deployment --replicas 5
    ```

3) Check pod status via `kubectl get pods`, as follows:

    **Command**
    ```
    $ kubectl get pods --show-labels --watch
    ```
    Observe **pending** status

4) Find out what's happening using `kubectl describe`, as follows:

    **Command**
    ```
    $ kubectl describe pod nginx-deployment-<ANY_PENDING_POD_ID>
    ```

    Here you can see the event generated by the scheduler saying that the Pod failed to schedule for reason FailedScheduling (and possibly others). The message tells us that there were not enough resources for the Pod on any of the nodes.


5) Check node status:

    **Command**
    ```sh
    $ kubectl describe nodes
    ```

    Think of alternatives to solve this...

5) Clean up:

    **Command**
    ```sh
    $ kubectl delete deployment nginx-deployment
    ```

---

[Back to Index](#index)

---

# Cluster-level monitoring

To scale an application and provide a reliable service, you need to understand how the application behaves when it is deployed. You can examine application performance in a Kubernetes cluster by examining the containers, pods, services, and the characteristics of the overall cluster. 

Kubernetes provides detailed information about an application’s resource usage at each of these levels. This information allows you to evaluate your application’s performance and where bottlenecks can be removed to improve overall performance.

---

## Exercise: kubectl top node
**Objectives:** Learn how to use `kubectl top node` to fetch metrics about nodes.

The easiest way to see basic node metrics is by using kubectl. The kubectl top command communicates with Metrics Server to get the latest node metrics and displays them in your terminal.

1) Deploy example pod

    **Command**
    ```
    $ kubectl apply -f workshop-07/manifests/nginx-with-request.yaml
    ```

2) Check node metrics

    **Command**

    ```
    $ kubectl top node

    NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
    minikube   137m         6%     730Mi           18%   
    ```

3) Clean up

    **Command**

    ```sh
    $ kubectl delete deployment nginx-deployment
    ```

---

[Back to Index](#index)

---

## Exercise: Kubernetes dashboard

[Kubernetes Dashboard](https://github.com/kubernetes/dashboard) is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage applications running in the cluster, troubleshoot them, as well as manage the cluster itself.

---

1) Start minikube dashboard:

    **Command**

    ```sh
    $ minikube dashboard
    ```

---

[Back to Index](#index)

---

## Exercise: Deploy weave scope

[Weave Scope](https://www.weave.works/docs/scope/latest/features/) is a visualization and monitoring tool for Docker and Kubernetes. It provides a top down view into your app as well as your entire infrastructure, and allows you to diagnose any problems with your distributed containerized app, in real time.

---

1) To install Weave Scope on your Kubernetes cluster, run:

    **Command**

    ```sh
    $ kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    ```

2) Open Scope in Your Browser:

    **Command**

    ```sh
    $ kubectl port-forward -n weave "$(kubectl get -n weave pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}')" 4040
    ```

3) Clean up:

    **Command**

    ```sh
    $ kubectl delete -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    ```

---

[Back to Index](#index)

---

## Exercise: Deploy stack elastic
**Objective:** Deploy the elastic stack with all its monitoring features.

---

**Note** make sure you run minikube with at least 4GB of memory

```sh
$ minikube start --memory 4096
```

### Create elastic namespace

**Command**

```sh
$ kubectl create ns elastic
```

Verify it's created

**Command**

```sh
$ kubectl get ns
```

### Deploy elastic logging stack from `workshop-06/manifests/elastic/` with the command below.

**Command**
```sh
$ kubectl create -Rf  workshop-06/manifests/elastic/elasticsearch
$ kubectl create -Rf  workshop-06/manifests/elastic/kibana
$ kubectl create -Rf  workshop-06/manifests/elastic/logstash
$ kubectl create -Rf  workshop-06/manifests/elastic/filebeat
```

Wait until the stack is ready

**Command**
```
$ kubectl get pods --show-labels --watch -n elastic
```

### Deploy kube state metrics

**Note** kube-state-metrics is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects. (See examples in the Metrics [section](https://github.com/kubernetes/kube-state-metrics/tree/master/docs#exposed-metrics). It is not focused on the health of the individual Kubernetes components, but rather on the health of the various objects inside, such as deployments, nodes and pods.

**Command**
```
$ kubectl apply -Rf workshop-07/manifests/elastic/kube-state-metrics
```

Check is up and running

**Command**
```
$  kubectl get pods --show-labels --watch -n kube-system
```

### Deploy metricbeat

**Command**

```sh
$ kubectl apply -Rf workshop-07/manifests/elastic/beats/metricbeat
```

Check is up and running

**Command**
```
$ kubectl get pods --show-labels --watch -n elastic
```

### Check Kibana

- Check index is created
- Check discover
- Check infra view (host, kubernetes, docker)
- Check [Metricbeat Kubernetes] Overview


**Command**
```
$ kubectl get pods --show-labels --watch -n elastic
```

### Enable port forwarding to kibana

**Command**
```
$ kubectl port-forward -n elastic  service/kibana 5601
```

Open your browser to ``localhost:5601``

### Deploy heartbeat

**Command**
```
$ kubectl apply -Rf workshop-07/manifests/elastic/beats/heartbeat
```

Check is up and running

**Command**
```
$ kubectl get pods --show-labels --watch -n elastic
```

### Check Kibana

- Check index is created
- Check discover
- Check uptime
- Check Heartbeat HTTP monitoring

# Cleaning up

Destroy the stack.

**Command**

```
$ kubectl delete -Rf workshop-06/manifests/elastic
$ kubectl delete -Rf workshop-07/manifests/elastic
$ kubectl delete ns elastic
```

---

[Back to Index](#index)

---
---

# Helpful Resources

* [elastic](https://www.elastic.co/products/)
* [metricbeat](https://www.elastic.co/guide/en/beats/metricbeat/6.8/index.html)
* [heartbeat](https://www.elastic.co/guide/en/beats/heartbeat/6.8/index.html)
* [packetbeat](https://www.elastic.co/guide/en/beats/packetbeat/6.8/index.html)

---

[Back to Index](#index)

---