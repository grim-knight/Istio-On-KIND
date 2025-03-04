# Istio-On-KIND

Istio Deployment on Mac M1/M2 with Kubernetes (Kind)

This guide will help you set up Istio with Kubernetes on Mac M1/M2 using **Kind** (Kubernetes in Docker).

Prerequisites

- Mac M1/M2
- Homebrew
- Docker Desktop

What Will Be Installed?

1. Docker
2. Kind (Kubernetes in Docker)
3. Kubectl
4. Istio Service Mesh
5. Bookinfo Application (Sample App)
6. Kiali Dashboard (Service Mesh Observability)
7. Jaeger (Distributed Tracing)
8. Prometheus (Metrics Monitoring)
9. Grafana (Visualization)
<!-- 10. Service Mesh Traffic Management (Canary Deployments, Fault Injection)
10. Mutual TLS Authentication (mTLS)
11. Authorization Policies
12. Rate Limiting
13. Circuit Breaking
14. External Service Egress Gateway
15. Custom Metrics with Prometheus
16. Access Logs Configuration
17. Policy Enforcement with OPA (Open Policy Agent) -->

# Tools Overview

### 1. Kiali (Service Mesh Observability)

- Visualizes the service mesh topology.
- Shows real-time traffic between services.
- Displays success rates, error rates, and request volume.
- Helps debug request routing and traffic management.

### 2. Jaeger (Distributed Tracing)

- Traces request paths across microservices.
- Helps visualize which services participated in a request.
- Useful for latency analysis and debugging performance issues.

### 3. Prometheus (Metrics Collection)

- Collects metrics from services and proxies.
- Monitors CPU, memory, and custom application metrics.
- Provides query language (PromQL) for metric analysis.

### 4. Grafana (Metrics Visualization)

- Visualizes metrics from Prometheus.
- Provides pre-configured dashboards for Istio.
- Custom dashboards can be built for app-specific metrics.

### 5. Service Mesh Traffic Management

- Canary Deployments: Route traffic percentages to different versions of a service.
- Fault Injection: Simulate failures to test resiliency.
- Circuit Breaking: Prevents cascading failures.

### 6. Mutual TLS (mTLS)

- Encrypts communication between services.
- Provides secure service-to-service authentication.

### 7. Authorization Policies

- Define fine-grained access control rules.
- Allow or deny traffic based on users, roles, or headers.

### 8. Rate Limiting

- Restrict the number of requests allowed per second.
- Helps protect services from abuse or overload.

### 9. Egress Gateway

- Controls outbound traffic to external services.
- Enforces security and policy rules on external communication.

### 10. Access Logs

- Logs detailed information about requests handled by Envoy proxies.

### Setup Steps

### 1. Install Docker

```bash
brew install --cask docker
open /Applications/Docker.app
```

Wait until Docker is running.

### 2. Install Kind and Kubectl

```bash
brew install kind kubectl
```

### 3. Create Kind Cluster

```bash
kind create cluster --name istio-cluster --config kind-istio-cluster.yaml
```

### 4. Install Istio CLI

```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH
```

To make this path permanent:

```bash
echo 'export PATH=$(pwd)/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
```

### 5. Install Istio in the Cluster

```bash
./bin/istioctl install --set profile=demo -y
kubectl create namespace istio-test
kubectl label namespace istio-test istio-injection=enabled
```

### 6. Deploy Bookinfo Application

```bash
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -n istio-test
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml -n istio-test
```

### 7. Install Kiali Dashboard

```bash
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system
```

### 8. Install Jaeger for Distributed Tracing

```bash
kubectl apply -f samples/addons/jaeger.yaml -n istio-system
kubectl rollout status deployment/jaeger -n istio-system
```

### 9. Install Prometheus for Metrics Monitoring

```bash
kubectl apply -f samples/addons/prometheus.yaml
kubectl rollout status deployment/prometheus -n istio-system
```

### 10. Install Grafana for Visualization

```bash
kubectl apply -f samples/addons/grafana.yaml
kubectl rollout status deployment/grafana -n istio-system
```

## What Does It Look Like

```
[User Request]
↓
[Istio Ingress Gateway]
↓
[Envoy Proxy] --> Metrics (Prometheus)
↓
[Bookinfo Microservices]
↓
Traces (Jaeger) | Logs | mTLS | Authorization
↓
Visualization (Grafana + Kiali)
```

## Run All the Different Tools

| Terminal   | Command                                                                 | URL                                         |
| ---------- | ----------------------------------------------------------------------- | ------------------------------------------- |
| Terminal 1 | `kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80` | [App](http://localhost:8080/productpage)    |
| Terminal 2 | `kubectl port-forward svc/kiali -n istio-system 20001:20001`            | [Kiali Dashboard](http://localhost:20001)   |
| Terminal 3 | `kubectl port-forward svc/tracing -n istio-system 16686:80`             | [Jaeger Tracing](http://localhost:16686)    |
| Terminal 4 | `kubectl port-forward svc/prometheus -n istio-system 9090:9090`         | [Prometheus Metrics](http://localhost:9090) |
| Terminal 5 | `kubectl port-forward svc/grafana -n istio-system 3000:3000`            | [Grafana Dashboard](http://localhost:3000)  |

### 11. Service Mesh Traffic Management

    Canary Deployments
    Deploy a virtual service to send 90% of traffic to `v1` and 10% to `v2`:

```bash
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-90-10.yaml -n istio-test
```

### Fault Injection

Simulate delays and aborts in service responses:

```bash
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml -n istio-test
```

Verify the fault injection through the **Bookinfo Application** and **Kiali Dashboard**.

### 12. Mutual TLS Authentication (mTLS)

Enable mutual TLS authentication between services for secure communication:
Apply strict mTLS policy:

```bash
kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml -n istio-test
```

Verify secure communication through Kiali by checking service-to-service TLS connections.

### 13. Authorization Policies

Define rules to allow or deny traffic between services:

Allow only `v1` of the `reviews` service to communicate with `ratings`:

```bash
kubectl apply -f samples/bookinfo/networking/authorization-policy.yaml -n istio-test
```

Verify through Kiali by observing denied traffic in the **Graph** view.

### 14. Rate Limiting

Apply a rate limit of 1 request per second on the `reviews` service:

```bash
kubectl apply -f samples/bookinfo/networking/virtual-service-ratelimit.yaml -n istio-test
```

Test the rate limiting by sending multiple requests and observing **429 Too Many Requests** responses.

### 15. Circuit Breaking

Prevent excessive retries to unhealthy services:

```bash
kubectl apply -f samples/bookinfo/networking/destination-rule-circuit-breaker.yaml -n istio-test
```

### 16. External Service Egress Gateway

    Route traffic to external services through Istio gateways:

```bash
kubectl apply -f samples/bookinfo/networking/egress-gateway.yaml -n istio-test
```

### 17. Custom Metrics with Prometheus

    Configure Prometheus to scrape custom application metrics.

### 18. Access Logs Configuration

    Enable detailed logs for Istio proxies:

```bash
kubectl apply -f samples/bookinfo/networking/envoy-access-logs.yaml -n istio-test
```

### 19. Policy Enforcement with OPA

    Apply advanced authorization policies:

```bash
kubectl apply -f samples/bookinfo/networking/opa-policy.yaml -n istio-test
```

Cleanup
To delete the cluster:

```bash
kind delete cluster --name istio-cluster
```

This guide is fully tested on Mac M1/M2 🚀
