# Workshop 10 - Ingress Controller


# Index
* [What is Ingress?](#what-is-ingress?)
  * [Exercise: Set up Ingress on Minikube with the NGINX Ingress Controller](#Exercise:-Set-up-Ingress-on-minikube-with-the-NGINX-ingress-controller)
  * [Exercise: Create fanout ingress-controlled app](#Exercise:-create-fanout-ingress-controlled-app)
  * [Exercise: Configuring Certificate-Based Authentication with Kubernetes Ingress-Nginx](#Exercise:-Configuring-Certificate-Based-Authentication-with-Kubernetes-Ingress-Nginx)

---

# What is Ingress?

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

```sh
        internet
            |
      [ Ingress ]
      --|-----|--
      [ Services ]
```

You must have an ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect.

You may need to deploy an ingress controller such as ingress-nginx. There are a number of ingress controllers you may choose from.

---

[Back to Index](#index)

---

## Exercise: Set up Ingress on Minikube with the NGINX Ingress Controller
**Objective** Install the nginx ingress controller in minikube via addon.

---

1. To enable the NGINX Ingress controller, run the following command:

    ```sh
    $ minikube addons enable ingress
    ```

2. Verify that the NGINX Ingress controller is `Running`

    ```sh
    $ kubectl get pods -n kube-system -w | grep ingress
    ```

    **Note**: This can take up to a minute.

---

[Back to Index](#index)

---

## Exercise: Create fanout ingress-controlled app
**Objective** To be able to create a fanout ingress access that redirects traffic between v1 and v2 of an example app.

### Deploy app (v1)

1. Create a Deployment using the following command:

    ```sh
    $ kubectl run web --image=gcr.io/google-samples/hello-app:1.0 --port=8080
    ```

2. Expose the Deployment:

    ```sh
    $ kubectl expose deployment web --target-port=8080 --type=NodePort
    ```
3. Verify the Service is created and is available on a node port:

    ```sh
    $ kubectl get service web
    ```

4. Visit the service via `NodePort`

    ```sh
    $ minikube service web --url
    ```

#### Create an Ingress resource

Let´s send traffic to your Service via hello-world.info.

1. Create the ingress

    **Command**

    ```sh
    $ kubectl apply -f workshop-10/manifests/example-ingress.yaml
    ```
2. Verify the IP address is set

    **Command**

    ```sh
    $ kubectl get ingress
    ```

3. Add the following line to the bottom of the /etc/hosts file.

    ```sh
    $ echo "$(minikube ip) hello-world.info" | sudo tee -a /etc/hosts
    ```

4. Test the ingress

    ```sh
    $ curl hello-world.info
    ```

### Deploy app (v2)

1. Create a v2 Deployment using the following command:
  
    ```sh
    $ kubectl run web2 --image=gcr.io/google-samples/hello-app:2.0 --port=8080
    ```

2. Expose the Deployment:

    ```sh
    $ kubectl expose deployment web2 --target-port=8080 --type=NodePort
     ```

### Update Ingress

1. Edit the existing `workshop-10/manifests/example-ingress.yaml` and add the following lines:

    ```yaml
    - path: /v2/*
       backend:
         serviceName: web2
         servicePort: 8080
    ```

2. Apply the changes:

      ```sh
      $ kubectl apply -f workshop-10/manifests/example-ingress.yaml
      ```

3. Test the accesses:

    ```sh
    $ curl hello-world.info
    ```

    ```sh
    $ curl hello-world.info/v2
    ```

## Clean up

  ```sh
  $ kubectl delete all -lrun=web
  $ kubectl delete all -lrun=web2
  $ kubectl delete ingress fanout-ingress
  ```
---

[Back to Index](#index)

---

## Exercise: Configuring Certificate-Based Authentication with Kubernetes Ingress-Nginx

SSL authentication (server --> client)

In SSL authentication, the client is presented with a server’s certificate, the client computer might try to match the server’s CA against the client’s list of trusted CAs. If the issuing CA is trusted, the client will verify that the certificate is authentic and has not been tampered with.

![alt text](workshop-10/images/1WaySSL.png)

### Setting Up Authentication

For this example we will be creating self-signed certificates. As a simple introduction, here are a couple of terms it would be useful to know:

- **CommonName(CN)**: Identifies the hostname or owner associated with the certificate.
- **Server Certificate**: A Certificate used to identify the server. The CN here is the hostname of the server. The Server Certificate is valid only if it is installed on a server where the hostname matches the CN.
  ```sh
  # Generate the Server Key and Certificate
  $ openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=hello-world.info"
  ```

  We should see this:

  ```sh
  Generating a 2048 bit RSA private key
  ....+++
  .....+++
  writing new private key to 'tls.key'
  ```

  And have these files created:

  ```sh
  .
  ├── tls.crt
  └── tls.key
  ```

### Creating the Kubernetes Secrets

We must store the certificates generated above in a Kubernetes Secret in order to use them in our Ingress-NGINX controller.

```sh
$ kubectl create secret tls tls-secret --key tls.key --cert tls.crt
```

Check is being properly created:

```sh
$ kubectl get secret tls-secret
```

### Deploy app

1. Deploy previous sample app

    **Command**
    ```sh
    $ kubectl run web --image=gcr.io/google-samples/hello-app:2.0 --port=8080
    ```
2. Expose the Deployment:

    ```sh
    $ kubectl expose deployment web --target-port=8080
     ```

3. Setup the Ingress Rules

    **Note** Make sure `hello-world.info` is already reachable via `/etc/hosts`

    Deploy the following:

    **Command**

    ```sh 
      $ kubectl apply -f workshop-10/manifests/certs/ingress.yaml
      ```

    This allows us to access the service `web` via `https://hello-world.info/`.

    - TLS is enabled and it is using the tls.key and tls.crt provided in the tls-secret secret.

    Check is properly created:

    **Command**
    ```sh
    kubectl get ing web-ingress-tls
    
    NAME          HOSTS              ADDRESS   PORTS     AGE
    web-ingress-tls   hello-world.info             80, 443   24s
    ```

---

[Back to Index](#index)

---

# Helpful Resources

* [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#types-of-ingress)
* [Nginx Ingress Controller](https://github.com/kubernetes/ingress-nginx)
* [Nginx Ingress Controller example](https://github.com/kubernetes/ingress-nginx/tree/master/docs/examples)
* [Nginx tls termination](https://kubernetes.github.io/ingress-nginx/examples/tls-termination/)

---

[Back to Index](#index)

---