```markdown
# Application Deployment and Traffic Management with Istio and Kubernetes

## Prerequisites

Ensure you have the following installed and configured on your local machine:

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Helm](https://helm.sh/docs/intro/install/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Skaffold](https://skaffold.dev/docs/install/)
- [Istioctl](https://istio.io/latest/docs/setup/getting-started/)

---

## Quick Start

This guide will walk you through setting up a local Kubernetes environment with Istio, installing observability tools, deploying an application, and configuring traffic management policies.

### 1. Start Minikube

Initialize Minikube with the necessary resources and plugins:

```bash
minikube start --memory=15970 --cpus=6 --network-plugin=cni --cni=calico
```

### 2. Build the Application

```bash
skaffold build
```

---

## Installing Istio and Observability Tools

### 3. Install Istio and Add-Ons

Use Istio's command-line interface to install Istio and observability add-ons:

```bash
istioctl install
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/kiali.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/grafana.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/prometheus.yaml
```

---

## Application Deployment with Helm

### 4. Install the Application

1. Create a namespace for the application and enable Istio injection:

    ```bash
    kubectl create namespace hipster-app
    kubectl label namespace hipster-app istio-injection=enabled
    ```

2. Install the application using Helm:

    ```bash
    cd helm-chart
    helm install onlineboutique . -n hipster-app
    ```

3. Apply Ingress Gateway configurations:

    ```bash
    cd ../istio-workshop
    kubectl apply -f addons-ingress.yaml
    kubectl apply -f frontend-ingress-gateway.yaml
    ```

---

## Traffic Management

### 5. Configure Traffic Splitting

To enable traffic splitting, apply the following configurations:

```bash
kubectl apply -f frontend-v2.yaml
kubectl apply -f frontend-virtualservice-split.yaml
kubectl apply -f frontend-destinationrule.yaml
```

---

## Istio Peer Authentication - mTLS

### 6. Enable Peer Authentication in Permissive Mode

```bash
kubectl apply -f hipster-app-mtls.yaml
```

### 7. Simulate Unencrypted Traffic Outside the Mesh

1. Create a test namespace and apply the Redis deployment:

    ```bash
    kubectl create namespace test
    kubectl apply -f redis.yaml
    ```

2. Access the Redis pod:

    ```bash
    # Inside the pod, run the following to check connectivity:
    wget frontend.hipster-app.svc.cluster.local
    ```

### 8. Enforce Strict Mode in Peer Authentication

Switch Peer Authentication to **Strict Mode** to block access outside the mesh:

1. Edit `hipster-app-mtls.yaml` to `STRICT` mode.
2. Access the Redis pod and try the connectivity again.
3. Update `redis.yaml` to set `sidecar.istio.io/inject: "true"`.

---

## Authorization Policies

### 9. Restrict Access to Redis-Cart Service

1. Apply a BusyBox deployment to test access to `redis-cart`:

    ```bash
    kubectl apply -f busybox.yaml
    ```

2. Access the BusyBox pod and run:

    ```bash
    telnet redis-cart 6379
    ```

### 10. Enforce Access Only from Specific Services

Configure authorization policies so only `cartservice` can access `redis-cart`:

1. Enable service accounts in `helm-chart/values.yaml`:

    ```yaml
    serviceAccount: cartservice
    ```

2. Upgrade the application in Helm:

    ```bash
    helm upgrade onlineboutique . -n hipster-app
    ```

3. Apply the authorization policy:

    ```bash
    kubectl apply -f authorization-policy.yaml
    ```

---

## Monitoring Egress Traffic

### 11. Monitor External Traffic in Kiali

1. Inside BusyBox, access an external site:

    ```bash
    wget www.google.com
    ```

2. Observe the logs in [Kiali](https://kiali.io/).

### 12. Restrict Egress Traffic

Change Istio’s configuration to restrict egress traffic:

```bash
istioctl install --set meshConfig.outboundTrafficPolicy.mode=REGISTRY_ONLY
```

Try accessing `www.google.com` from BusyBox to confirm restriction.

### 13. Allow Specific Egress Traffic

To allow traffic only to specific sites, create a ServiceEntry for `google.com`:

```bash
kubectl apply -f google-serviceentry.yaml
```

---

## Network Tunnel for Access

To access services on Minikube from your host machine:

1. Edit `/etc/hosts`:

    ```plaintext
    127.0.0.1       control.test
    127.0.0.1       app.test
    ```

2. Create a Minikube network tunnel:

    ```bash
    minikube tunnel -p profile_name
    ```

---

This structured approach provides a guided setup for managing applications with Istio, observability tools, and traffic policies in a local Kubernetes environment.
``` 

You can copy this complete README content directly into your `README.md` file.
