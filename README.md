# Kubernetes Voting Application

A multi-service voting application deployed on Kubernetes to understand how real distributed systems work.

This isn't a tutorial project. It's a learning project focused on Kubernetes networking, service discovery, and ingress routing.

## What This Project Does

Users can vote between two options. Votes get queued, processed asynchronously, and displayed in real-time on a results page.

The application has five independent services:
- **Voting Frontend** - where users cast votes
- **Results Frontend** - where users view results
- **Redis** - acts as a message queue
- **PostgreSQL** - persistent storage
- **Worker** - processes votes asynchronously

Each service runs in its own container and communicates through Kubernetes Services.

## Why This Architecture

This mirrors real microservices systems:
- Frontend services are stateless and horizontally scalable
- Data services are isolated for persistence
- All communication happens via service discovery
- External traffic enters through a single controlled entry point

Kubernetes handles orchestration, self-healing, and networking.

## What I Learned

Before this project, I could write Kubernetes manifests. After this project, I understand how Kubernetes actually routes traffic.

Key lessons:
- Kubernetes networking is service-driven, not pod-driven
- Ingress requires both rules and a controller
- Service DNS resolution is namespace-scoped
- Local clusters behave differently than cloud clusters
- Pod IPs are ephemeral, Services are stable

## Architecture

```
External Traffic
      ↓
Ingress Controller
      ↓
┌─────────────────────────────────────────┐
│         Kubernetes Cluster              │
│                                         │
│  Voting Frontend ←→ Redis               │
│                      ↕                  │
│  Results Frontend ←→ PostgreSQL ←→ Worker │
│                                         │
└─────────────────────────────────────────┘
```

All internal communication uses ClusterIP Services.
External access goes through Ingress with host-based routing.

## Kubernetes Objects Used

- **Deployments** - manage application workloads and replica counts
- **Pods** - smallest schedulable units (ephemeral, auto-recreated)
- **Services (ClusterIP)** - stable internal networking
- **Services (NodePort)** - used temporarily for testing
- **Ingress** - HTTP routing rules for external traffic
- **Ingress Controller** - processes traffic based on Ingress rules

## Traffic Flow

### Internal
- Voting frontend → Redis (via Service name)
- Worker → Redis (via Service name)
- Worker → PostgreSQL (via Service name)
- Results frontend → PostgreSQL (via Service name)

No pod IPs used anywhere. Service DNS is resolved automatically.

### External
1. User sends request from browser
2. Request hits Ingress Controller
3. Ingress rules evaluate host/path
4. Traffic forwards to correct Service
5. Service load-balances to backend pods

## Prerequisites

- Kubernetes cluster (minikube, kind, k3s, or cloud cluster)
- kubectl configured
- Ingress controller installed (nginx-ingress or similar)

## Setup

Clone the repository:
```bash
git clone https://github.com/yourusername/kubernetes-voting-app.git
cd kubernetes-voting-app
```

Deploy the application:
```bash
kubectl apply -f k8s/
```

Verify all pods are running:
```bash
kubectl get pods
```

Check services:
```bash
kubectl get svc
```

Verify ingress:
```bash
kubectl get ingress
```

## Local Access

For local clusters, you'll need to expose the Ingress Controller.

Using port forwarding:
```bash
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80
```

Add hostname to `/etc/hosts`:
```
127.0.0.1 vote.local result.local
```

Access the application:
- Voting: http://vote.local:8080
- Results: http://result.local:8080

## What Broke and How I Fixed It

### Pod IPs kept changing
**Problem:** Pods get recreated, IPs change  
**Solution:** Use Services as stable endpoints

### Ingress didn't work initially
**Problem:** Created Ingress resources but traffic wasn't reaching apps  
**Solution:** Installed Ingress Controller separately

### Ingress Controller wouldn't schedule
**Problem:** Controller pod stuck in pending state  
**Solution:** Fixed node labels and tolerations for local cluster

### Service names didn't resolve across namespaces
**Problem:** DNS resolution scope wasn't clear  
**Solution:** Used fully qualified domain names where needed

## Project Structure

```
.
├── k8s/
│   ├── voting-deployment.yaml
│   ├── voting-service.yaml
│   ├── result-deployment.yaml
│   ├── result-service.yaml
│   ├── redis-deployment.yaml
│   ├── redis-service.yaml
│   ├── postgres-deployment.yaml
│   ├── postgres-service.yaml
│   ├── worker-deployment.yaml
│   └── ingress.yaml
└── README.md
```

## Key Takeaways

This project isn't about running containers. It's about understanding how Kubernetes:
- Routes traffic between services
- Discovers services dynamically
- Separates internal and external networking
- Enforces declarative state

Once this mental model clicked, advanced Kubernetes topics started making sense.

## Cleanup

Remove all resources:
```bash
kubectl delete -f k8s/
```

## Related

Full writeup on dev.to: []
