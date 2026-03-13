# Kubernetes HTTPS Demo App (Port Forward + Ingress)

This project demonstrates how to expose a simple application over **HTTPS in Kubernetes** using a TLS certificate.
The application is deployed in a **Kubernetes cluster** and accessed through an **Ingress resource**.
For testing in a local or restricted environment, **port forwarding** is used to access the application from outside the cluster.

---

## Architecture

```
Browser
   │
HTTPS :8080
   │
kubectl port-forward
   │
Ingress
   │
Service
   │
Pod (Demo App)
```

Traffic Flow:

1. User sends HTTPS request from the browser.
2. The request reaches the local machine on port **8080**.
3. `kubectl port-forward` forwards traffic to the **Ingress controller**.
4. The Ingress routes the request to the **Kubernetes Service**.
5. The service sends traffic to the **application pod**.

---

## Prerequisites

Make sure the following tools are installed:

* Kubernetes cluster
* kubectl
* Docker
* OpenSSL
* Ingress controller enabled in the cluster

Example environments:

* Minikube
* Cloud VM (e.g., EC2 instance)

---

## Step 1 — Create Demo Application

Create a simple HTML file.

`index.html`

```
<h1>DevOps SSL Demo App</h1>
<p>Application running with HTTPS in Kubernetes</p>
```

---

## Step 2 — Create Deployment

`deployment.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-app
        image: nginx
        ports:
        - containerPort: 80
```

Apply the deployment:

```
kubectl apply -f deployment.yaml
```

---

## Step 3 — Create Service

`service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: demo-service
spec:
  selector:
    app: demo-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Apply the service:

```
kubectl apply -f service.yaml
```

---

## Step 4 — Generate TLS Certificate

Generate a self-signed certificate using OpenSSL.

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout tls.key \
-out tls.crt
```

---

## Step 5 — Create Kubernetes TLS Secret

```
kubectl create secret tls demo-tls \
--cert=tls.crt \
--key=tls.key
```

Verify the secret:

```
kubectl get secrets
```

---

## Step 6 — Create Ingress with TLS

`ingress.yaml`

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
spec:
  tls:
  - secretName: demo-tls

  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: demo-service
            port:
              number: 80
```

Apply the ingress:

```
kubectl apply -f ingress.yaml
```

Verify:

```
kubectl get ingress
```

---

## Step 7 — Port Forward the Ingress

Forward port **8080** from the local machine to the ingress controller.

Example command:

```
kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx 8080:443
```

This forwards:

```
Local Port 8080 → Ingress HTTPS Port 443
```

---

## Step 8 — Access the Application

Open the browser or use curl:

```
https://EC2_PUBLIC_IP:8080
```

Example:

```
https://3.110.77.244:8080
```

Or test using curl:

```
curl -k https://3.110.77.244:8080
```

The `-k` flag allows curl to ignore certificate verification since a self-signed certificate is used.

---

## Expected Output

```
DevOps SSL Demo App
Application running with HTTPS in Kubernetes
```

---

## Notes

* The browser may show **“Not Secure”** because the certificate is self-signed.
* This setup is intended for **testing and learning purposes**.
* In production environments, certificates should be issued by a trusted Certificate Authority.

---

## Production Setup (Recommended)

In real Kubernetes environments, HTTPS is typically automated using:

* cert-manager
* Let's Encrypt certificates
* Ingress controllers
* External load balancers

This enables:

* automatic certificate generation
* automatic renewal
* trusted HTTPS connections

---

## Useful Commands

Check pods:

```
kubectl get pods
```

Check services:

```
kubectl get svc
```

Check ingress:

```
kubectl get ingress
```

Check secrets:

```
kubectl get secrets
```

---

## Summary

This demo shows how to:

* Deploy an application in Kubernetes
* Expose it through a service
* Secure it using TLS
* Route traffic using Ingress
* Access the application using port forwarding
* Test HTTPS connectivity using curl

This setup is useful for **DevOps practice and Kubernetes networking concepts**.
