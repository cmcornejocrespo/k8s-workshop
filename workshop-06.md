# Workshop 06 - Logging Architecture

Application and systems logs can help you understand what is happening inside your cluster. The logs are particularly useful for debugging problems and monitoring cluster activity.

Kubernetes provides no native storage solution for log data, but you can integrate many existing logging solutions into your Kubernetes cluster.

This tutorial covers the fundamental building blocks that make up Kubernetes. Understanding what these components are
and how they are used is crucial to learning how to use the higher level objects and resources.

# Index
* [Basic logging](#Basic-logging)
  * [Exercise: Basic logging](#Exercise:-Basic-logging)
* [Cluster-level logging](#Cluster-level-logging)
  * [Exercise: Cluster-level logging](#Exercise:-Deploy-ELKF)
* [Cleaning up](#cleaning-up)
* [Helpful Resources](#helpful-resources)

---

# Basic logging
This demonstration uses a pod specification with a container that writes some text to standard output once per second.

---

## Exercise: Basic logging
**Objectives:** Learn how to access logs via `kubectl`.

---

1) Deploy example pod
```
$ kubectl apply -f workshop-06/manifests/counter-pod.yaml
```

2) To fetch the logs, use the `kubectl` logs command, as follows:
```
$ kubectl logs counter
```
3) To fetch the logs following:
```
$ kubectl logs counter -f
```
4) Clean up:
```
$ kubectl delete counter
```
**note** You can use kubectl logs to retrieve logs from a previous instantiation of a container with --previous flag, in case the container has crashed.

**note** If your pod has multiple containers, you should specify which container’s logs you want to access by appending a container name to the command

```
# Begin streaming the logs of the ruby container in pod web-1
$ kubectl logs -f -c ruby web-1
```

---

[Back to Index](#index)

---

# Cluster-level logging

You can implement cluster-level logging by including a node-level logging agent on each node. The logging agent is a dedicated tool that exposes logs or pushes logs to a backend. Commonly, the logging agent is a container that has access to a directory with log files from all of the application containers on that node.

We'll deploy a cluster-level logging solution based on ELK+F (elasticsearch, logstash, kibana and filebeat).

---

## Exercise: Deploy ELKF
**Objective:** Deploy the stack anf use Kibana to examine the logs from an example pod.

---
1) Create a simple Pod called `counter` using the `busybox` image. Use the
manifest `workshop-06/manifests/counter-pod.yaml` or the yaml below.

**workshop-06/manifests/counter-pod.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
```

**Command**
```
$ kubectl create -f workshop-06/manifests/counter-pod.yaml
```

2) Create namespace called `elastic`. 

**Command**
```
$ kubectl create ns elastic
```

Verify it's created

**Command**
```
$ kubectl get ns
```

3) Deploy elasticsearch, Use the
manifests inside `workshop-06/manifests/elastic/elasticsearch`.

**Command**
```
$ kubectl apply -Rf workshop-06/manifests/elastic/elasticsearch
```

Check is up and running

**Command**
```
$ kubectl get pods --show-labels --watch -n elastic
```

4) Deploy Kibana, Use the
manifests inside `workshop-06/manifests/elastic/kibana`.

**Command**
```
$ kubectl apply -Rf workshop-06/manifests/elastic/kibana
```

Check is up and running

**Command**
```
$ kubectl get pods --show-labels --watch -n elastic
```

5) Deploy Logstash, Use the
manifests inside `workshop-06/manifests/elastic/logstash`.

**Command**
```
$ kubectl apply -Rf workshop-06/manifests/elastic/logstash
```

Check is up and running

**Command**
```
$ kubectl get pods --show-labels --watch -n elastic
```

6) Enable port forwarding to kibana.

**Command**
```
$ kubectl port-forward -n elastic  service/kibana 5601
```

Open your browser to ``localhost:5601``

7) Deploy Filebeat, Use the
manifests inside `workshop-06/manifests/elastic/filebeat`.

**Command**
```
$ kubectl apply -Rf workshop-06/manifests/elastic/filebeat
```

Check is up and running

**Command**
```
$ kubectl get pods --show-labels --watch -n elastic
```

8) Configure index in Kibana and demo the stack

# Cleaning up

Destroy the stack.

**Command**
```
$ kubectl delete -Rf workshop-06/manifests/elastic
$ kubectl delete ns elastic
```

---

[Back to Index](#index)

---
---

# Helpful Resources

* [kubectl logs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs)
* [elastic](https://www.elastic.co/products/)

---

[Back to Index](#index)

---