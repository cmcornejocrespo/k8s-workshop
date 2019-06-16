# Workshop 09 - Helm the Kubernetes package manager


# Index
* [What is Helm?](#What-is-Helm?)
* [Using helm](#Using-helm)
  * [Exercise: Installing Helm](#Exercise:-Installing-Helm)
  * [Exercise: Helm Repos](#Exercise:-Helm-Repos)
  * [Exercise: Managing the life cycle of my first release](#Exercise:-Managing-the-life-cycle-of-my-first-release)
* [Exercise: Creating your first helm app](#Exercise:-creating-your-first-helm-app)

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

To add new chart repos you can use `helm repo add`

**Command**

```sh
helm repo add bitnami https://charts.bitnami.com  
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

As you see, Kubernetes has created a bunch of resources for your Redis app. If you are using   `Minikube`, open the Kubernetes dashboard with minikube dashboard  to see them or use kubectl get commands to display them in your terminal.

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

## Exercise: Creating your first helm app
**Objectives:** Our goal for this workshop is to create a Helm chart that takes the voting app demo and installs it in a Kubernetes cluster.

Our application has five parts:

1. A front-end for users to vote
2. A back-end that tallies votes
3. An admin interface to see the results
4. A Redis cache
5. A PostgreSQL database

![alt text](workshop-09/images/voting-app.png)

### Deploy voting app

1) Deploy the app via manifests

    **Command**
    ```sh
    $ kubectl create namespace vote
    ```

    **Command**
    ```sh
    $ kubectl apply -Rf workshop-09/manifests/voting-app
    ```

    Wait for the app to start

    ```sh
    kubectl get pods -n vote -w
    ```

    The vote interface is then available on port `31000` on each host of the cluster, the result one is available on port `31001`

    Delete the app

    **Command**

    ```sh
    $ kubectl delete -Rf workshop-09/manifests/voting-app
    ```

### Create voting app chart

1) Charts usage recap

    Helm uses charts as the installable unit. A Chart is a package containing at least three components:

    - A Chart.yaml, which describes the chart
    - Templates, which Helm transforms into Kubernetes manifests
    - A values.yaml file, which defines and describes configurable parameters

    These files are all bundled together into an archive that can be moved from one place to another

2) Creating a Chart

    Start by running the Helm command to create a new basic chart:

    ```sh
    $ helm create voter
    ```

    It should looks like

    ```sh
    ├── Chart.yaml
    ├── charts
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   ├── ingress.yaml
    │   ├── service.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml
    ```

    We will start by modifying the Chart.yaml

    ```yaml
    apiVersion: v1      # The chart schema version, always v1 for Helm 2
    version: 0.1.0      # The version of the chart. Change this for each release.
    appVersion: "1.0"   # The version of the main app in this chart. OPTIONAL
    description: An example cloud native voting app
    name: voter
    ```

    You can verify that you correctly entered the data by running:

    ```sh
    $ helm inspect chart ../voter 
    ```

3) Templating Charts recap

    Helm charts contain templates, which at its core are resource descriptions that Kubernetes can understand.
    
    The Helm template language is written in `gotemplate`, a templating language provided by the Go standard library. Templates can be rendered locally for debugging purposes using the `helm template` command.

    In the last section, we created a Helm chart that contained three templates:

    In the last section, we created a Helm chart that contained three templates:

    - A Deployment for running a microservice (deployment.yaml)
    - A Service for routing traffic to that microservice (service.yaml)
    - An Ingress that can optionally be turn on to allow external traffic to access your app (ingress.yaml)

    Let's take a look at a snippet of the Deployment template created in an earlier section:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ include "voter.fullname" . }}
      labels:
    {{ include "voter.labels" . | indent 4 }}
    spec:
      replicas: {{ .Values.replicaCount }}
      selector:
        matchLabels:
          app.kubernetes.io/name: {{ include "voter.name" . }}
          app.kubernetes.io/instance: {{ .Release.Name }}
      template:
        metadata:
          labels:
            app.kubernetes.io/name: {{ include "voter.name" . }}
            app.kubernetes.io/instance: {{ .Release.Name }}
        spec:
        {{- with .Values.imagePullSecrets }}
          imagePullSecrets:
            {{- toYaml . | nindent 8 }}
        {{- end }}
          containers:
            - name: {{ .Chart.Name }}
              image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
              imagePullPolicy: {{ .Values.image.pullPolicy }}
              ports:
                - name: http
                  containerPort: 80
                  protocol: TCP
    ```

    As we can see, this looks a bit like a Deployment object in YAML format, sprinked with a few templates from gotemplate. We can see a few unique templates. Let's look at a few in particular:

    1. `{{ template "voter.fullname" . }}`
    2. `{{ template "voter.name" . }}`
    3. `{{ .Release.Name }}`
    4. `{{ .Values.replicaCount }}`

    template `voter.fullname` is a template partial. Template partials are `helper` functions defined within the chart to abstract some common funcion calls to make the templates more easily readable and DRY. When we first created the chart, a file called `_helpers.tpl` was created in the templates directory. That file is the default location for template partials, and Helm provides a few template partials out the gate in that file.

    `voter.fullname` accepts one variable as input: the dot variable (.). The dot variable contains all of the variables injected by Helm into the template engine, such as values or information about the cluster.

    The output of `voter.fullname` is the chart name (voter) and the release name, joined with a hyphen (voter-release-name).

    `voter.name` is just the chart name, so `voter`.

    **In a nutshell**, When you provide the following in values.yaml:

    ```yaml
    image:
      repository: dockersamples/examplevotingapp_vote
    ```

    Helm will render that as `.Values.image.repository` to be made available to you in the templates.

    Run `helm template ./voter` to view the chart fully rendered as YAML manifests. This is what will eventually be supplied to Kubernetes.

4) Configuring the Front-End (vote)

    Take a look at the `values.yaml` file generated for us.
    
    This default values file has standard configurable parameters pre-defined. These parameters are used by files in templates.

    The default Helm chart defines three Kubernetes resources:

    - A Deployment for running a microservice
    - A Service for routing traffic to that microservice
    - An Ingress that can optionally be turn on to allow external traffic to access your app

    It just so happens that we need these things for our example. So to get started, we'll change the default values to make use of them.

    In the images section, make the following changes:

    - Point repository to `dockersamples/examplevotingapp_vote`
    - Set tag to `before`

    And also change one thing in service:

    - port: `5000`
    - type: `NodePort`
    - nodePort: `31000`


5) Adding Templates for the rest of the voting-app

    Copy from workshop-09/manifests/voting-app/ into /templates(Remember to make sure that the name is unique for each installation.)

    Try to creata a similar filestructure

    ```sh
    ├── Chart.yaml
    ├── templates
    │   ├── _helpers.tpl
    │   ├── db
    │   │   ├── db-deployment.yaml
    │   │   └── db-service.yaml
    │   ├── redis
    │   │   ├── redis-deployment.yaml
    │   │   └── redis-service.yaml
    │   ├── result
    │   │   ├── result-deployment.yaml
    │   │   └── result-service.yaml
    │   ├── vote
    │   │   ├── vote-deployment.yaml
    │   │   └── vote-service.yaml
    │   └── worker
    │       └── worker-deployment.yaml
    └── values.yaml
    ```

    **Tip** You can follow this values.yaml as example to create all the templates

    ```yaml
    # Default values for voter.
    # This is a YAML-formatted file.
    # Declare variables to be passed into your templates.

    # Voting app
    vote:
      replicaCount: 1

      image:
        repository: dockersamples/examplevotingapp_vote
        tag: before
        pullPolicy: IfNotPresent

      imagePullSecrets: []
      nameOverride: ""
      fullnameOverride: ""

      service:
        type: NodePort
        port: 5000
        targetPort: 80
        nodePort: 31000

      ingress:
        enabled: false
        annotations: {}
          # kubernetes.io/ingress.class: nginx
          # kubernetes.io/tls-acme: "true"
        hosts:
          - host: chart-example.local
            paths: []

        tls: []
        #  - secretName: chart-example-tls
        #    hosts:
        #      - chart-example.local

      resources: {}
        # We usually recommend not to specify default resources and to leave this as a conscious
        # choice for the user. This also increases chances charts run on environments with little
        # resources, such as Minikube. If you do want to specify resources, uncomment the following
        # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
        # limits:
        #   cpu: 100m
        #   memory: 128Mi
        # requests:
        #   cpu: 100m
        #   memory: 128Mi

      nodeSelector: {}

      tolerations: []

      affinity: {}

    result:
      replicaCount: 1
      image:
        repository: dockersamples/examplevotingapp_result
        tag: before
        pullPolicy: IfNotPresent

      service:
        type: NodePort
        port: 5001
        targetPort: 80
        nodePort: 31001

    worker:
      replicaCount: 1
      image:
        repository: dockersamples/examplevotingapp_worker
        pullPolicy: IfNotPresent

    redis:
      replicaCount: 1
      image:
        repository: redis
        tag: alpine
        pullPolicy: IfNotPresent

      service:
        type: ClusterIP
        port: 6379

    db:
      replicaCount: 1
      image:
        repository: postgres
        tag: 9.4
        pullPolicy: IfNotPresent

      service:
        type: ClusterIP
        port: 5432
    ```

6) Installing the chart

    The first time you can install the chart as follows (selecting in which namespace you want)

    ```sh
    helm install -n voting-app ./voter --namespace=vote
    ```

    On every change of the chart you could run the following:

    **Command**

    ```sh
    helm upgrade voting-app ./voter --namespace=vote
    ```
    
8) Adding existing chart as dependencies

    We were building a chart based on a sample voting app. Here are the images we needed:

    - redis:alpine - The Redis server for a work queue
    - postgres:9.4 - The persistent data storage
    - dockersamples/examplevotingapp_result:before (The admin viewer)
    - dockersamples/examplevotingapp_vote:before (The voting frontend)
    - dockersamples/examplevotingapp_worker (The queue worker)

    Let´s find out if we have any redis or postgres available as charts
    ```sh
    $ helm search | grep redis
    bitnami/redis                                   8.0.8           5.0.5                           Open source, advanced key-value store. It is often referr...
    incubator/redis-cache                           0.4.1           4.0.12-alpine                   A pure in-memory redis cache, using statefulset and redis...
    stable/prometheus-redis-exporter                2.0.0           0.28.0                          Prometheus exporter for Redis metrics                       
    stable/redis                                    8.0.8           5.0.5                           Open source, advanced key-value store. It is often referr...
    stable/redis-ha                                 3.5.0           5.0.3                           Highly available Kubernetes implementation of Redis 
    ```

    **Command**
    ```sh
    $ helm search | grep postgres
    bitnami/postgresql                              5.3.8           11.3.0                          Chart for PostgreSQL, an object-relational database manag...
    stable/postgresql                               5.3.8           11.3.0                          Chart for PostgreSQL, an object-relational database manag...
    stable/prometheus-postgres-exporter             0.6.2           0.4.7                           A Helm chart for prometheus postgres-exporter 
    ```

    Looks like we need to take care only of the dockersamples charts.

8.1) Subcharts and `requirements.yaml`

    In Helm, a subchart is a regular Helm chart that is a dependency of your chart. By adding it to a special file called `requirements.yaml`, you can let Helm manage that chart for you.

    Let's create a `requirements.yaml` file next to the `Chart.yaml` in our chart. Here's what belongs inside:

    ```yml
    dependencies:
    - name: redis        # from search results above
      version: 8.0.8     # Also from the search results above
      repository: https://kubernetes-charts.storage.googleapis.com
    ```

    **Note** that the repository URL comes from running helm repo list and copying the URL for the stable repo, which is where the Redis chart lives.

    Next, run helm search postgresql and repeat the above for Postgres. By the end of that step, you should have two items in your dependencies array in requirements.yaml

8.2) Download the Dependencies
    
    Run helm dep up to have Helm fetch those dependencies for you. When completed, you should be able to see two new directories inside of your chart's charts subdirectory.

    ```sh
    $ helm dependency update
    ```

    Your folder should look like this:

    ```sh
    ── Chart.yaml
    ├── charts
    │   ├── postgresql-5.3.8.tgz
    │   └── redis-8.0.8.tgz
    ├── requirements.lock
    ├── requirements.yaml
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   ├── ingress.yaml
    │   ├── service.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml
    ```

    Next, we'll start working with Helm templates

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