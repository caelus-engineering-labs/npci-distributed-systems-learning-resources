# Code Repositories

Twelve distributed-systems demo projects, each packed into a single `.bundle` file with its full history: every branch, every commit, every tag. You don't need access to the original GitHub repositories, which are private; the bundle *is* the repository.

The projects are listed below from the simplest to the most complex. If you're working through them on your own, go in that order.

---

## What a bundle is

A git bundle is a whole repository in one file. Git can clone from it exactly as it clones from GitHub, so once you've cloned a bundle you have a normal git repository: you can read the history, switch branches and commit your own work.

## What you need

- **Git**, any recent version (`git --version`).
- **A bash shell:** macOS, Linux, or Git Bash/WSL on Windows.
- **For the Kubernetes projects:** a local single-node cluster (Docker Desktop with Kubernetes turned on, minikube, OrbStack or kind) and `kubectl`.
- **For the Docker projects:** Docker with Compose v2 (Docker Desktop, OrbStack, Rancher Desktop or Docker Engine).
- **For the Go projects** (scatter-gather and resilience patterns): Go 1.24 or later. The other Go projects build inside Docker, so they don't need Go installed.

Each project's section below says which of these it needs, and its own `README.md` has the details.

## How to use a bundle

These steps use `resilience-patterns-in-go` as the example. Every project works the same way; its own commands are in its section further down.

### 1. Get the bundles

Download this repository (the green **Code** button, then **Download ZIP**, and unzip it), or clone it:

```bash
git clone https://github.com/caelus-engineering-labs/npci-distributed-systems-learning-resources.git
cd npci-distributed-systems-learning-resources/code-repositories
```

### 2. Clone the project from its bundle

```bash
git clone resilience-patterns-in-go.bundle
cd resilience-patterns-in-go
```

This makes a folder named after the bundle, with the project's default branch checked out (`main` here).

### 3. See its branches

```bash
git branch -r
```

You'll see one `origin/<branch>` line per branch: every branch is already in your clone.

### 4. Switch to a branch

```bash
git switch 03-circuit-breaker
```

The first time, this turns `origin/03-circuit-breaker` into a local branch of the same name and checks it out. Switch back with `git switch main`.

To clone straight onto a branch instead, use `-b`:

```bash
git clone -b 03-circuit-breaker resilience-patterns-in-go.bundle
```

To see what a branch changed compared with another one:

```bash
git diff main 03-circuit-breaker
```

### 5. Follow the project's own README

Open the project's `README.md`. Some of them start with a `git clone https://github.com/...` step: **skip it**: you already have the code from the bundle, and those GitHub repositories are private. Run the remaining commands from inside the folder you cloned.

### Check a bundle (optional)

From inside any git repository (for example, a project you've just cloned):

```bash
git bundle verify ../resilience-patterns-in-go.bundle
```

It should list the bundle's branches and say `The bundle records a complete history.`

### Keep your own copy on GitHub (optional)

Your clone's `origin` points at the bundle file. To push your work to a repository of your own:

```bash
git remote set-url origin https://github.com/<you>/<repository>.git
git push -u origin --all
git push origin --tags
```

Pushing every branch this way needs a local copy of each one, so `git switch` to each branch first.

---

## The projects, from simplest to most complex

Clean up after each project, so the next one starts on a quiet machine. The Kubernetes projects that use `kubectl apply -f .` are removed with `kubectl delete -f .` from the same folder, and the ones with a `demo.sh` with `./demo.sh down`. For `k8s-distributed-caching`, run `kubectl delete -f infinispan.yaml`. The Docker projects are removed with `docker compose down` or `./demo.sh down`. Each project's README has the exact command.

### 1. k8s-pv-pvc-statefulset-mysql

**What it shows:** PersistentVolumes, PersistentVolumeClaims and StatefulSets, with one MySQL server. The demo kills its pod, and the replacement picks up the same disk and carries on: the data survives. (This is about durability, not high availability.)

**Needs:** a Kubernetes cluster with a default StorageClass.

**Branches:** `main`

```bash
git clone k8s-pv-pvc-statefulset-mysql.bundle
cd k8s-pv-pvc-statefulset-mysql
./demo.sh
```

### 2. k8s-load-balancing

**What it shows:** three ways to spread requests over three nginx pods. Branch `01` uses a Service (kube-proxy, layer 4: one pick per TCP connection), `02` an nginx proxy (layer 7: one pick per HTTP request), and `03` lets the client choose the pod itself.

**Needs:** a Kubernetes cluster (the README uses minikube).

**Branches:** `01-l4-load-balancer` (default), `02-l7-load-balancer`, `03-client-side-load-balancer`. This project has no `main` branch.

```bash
git clone k8s-load-balancing.bundle
cd k8s-load-balancing
git switch 02-l7-load-balancer
```

### 3. k8s-log-aggregation

**What it shows:** the log aggregation pattern. Five nginx pods write logs; Grafana Alloy collects them on every node and pushes them to Loki; you search them in Grafana.

**Needs:** a Kubernetes cluster (the README uses minikube).

**Branches:** `main`

```bash
git clone k8s-log-aggregation.bundle
cd k8s-log-aggregation
kubectl apply -f .
```

### 4. k8s-api-gateway

**What it shows:** the API gateway pattern with the Kubernetes Gateway API and Envoy Gateway. One entry point routes `/api/users` and `/api/orders` to two hidden services. Branch `02-bff` adds the backend-for-frontend pattern: a separate backend for the web app and for the mobile app, each calling both services and returning what its screen needs.

**Needs:** a Kubernetes cluster (the README uses minikube) and internet access to install Envoy Gateway.

**Branches:** `main`, `02-bff`

```bash
git clone k8s-api-gateway.bundle
cd k8s-api-gateway
git switch 02-bff
```

### 5. k8s-distributed-caching

**What it shows:** a three-node Infinispan cache. A distributed cache keeps two copies of each key spread across the nodes, survives losing a node, and rebalances when you scale; a replicated cache keeps every key on every node. `demo.sh` has one command per step.

**Needs:** a Kubernetes cluster.

**Branches:** `main`

```bash
git clone k8s-distributed-caching.bundle
cd k8s-distributed-caching
kubectl apply -f infinispan.yaml
```

### 6. postgresql-ha-in-k8s

**What it shows:** PostgreSQL high availability with the CloudNativePG operator: three instances, a client that keeps writing, and a killed primary that the cluster recovers from on its own.

**Needs:** a Kubernetes cluster and internet access to install the operator.

**Branches:** `main`

```bash
git clone postgresql-ha-in-k8s.bundle
cd postgresql-ha-in-k8s
./demo.sh
```

The `postgresql-ha-in-k8s-main.zip` at the top of this repository is an older version of this project; use the bundle.

### 7. scatter-gather-pattern

**What it shows:** the scatter-gather pattern in Go, as a price comparison service. The aggregator asks four vendors at once, keeps whatever arrives within 300 ms and returns the cheapest quote, so slow or broken vendors make the answer less complete but never late. Standard library only.

**Needs:** Go.

**Branches:** `main`

```bash
git clone scatter-gather-pattern.bundle
cd scatter-gather-pattern
./demo.sh
go test -race ./...
```

### 8. saga-patterns-study

**What it shows:** the saga pattern, as four Go services (order, payment, inventory, shipping) completing an order over a NATS bus. Branch `01` is choreography: services only react to each other's events. Branch `02` is orchestration: the order service directs each step. Every tenth order fails at inventory, and the saga compensates by refunding the payment.

**Needs:** Docker with Compose.

**Branches:** `01-choreography-pattern` (default), `02-orchestration-pattern`. This project has no `main` branch.

```bash
git clone saga-patterns-study.bundle
cd saga-patterns-study
docker compose up -d --build
./demo.sh
docker compose down
git switch 02-orchestration-pattern
```

### 9. kafka-event-sourcing

**What it shows:** event sourcing with Kafka. Ten years of payroll for 100 employees are kept only as events (hires, raises, monthly payments), and salaries, totals, histories and the payroll on any past date are all rebuilt by replaying them.

**Needs:** Docker with Compose. Go is not needed; the program is built inside Docker.

**Branches:** `main`. **Tags:** `v1.0.0` (an earlier design), `v2.0.0` (the current payroll version).

```bash
git clone kafka-event-sourcing.bundle
cd kafka-event-sourcing
./demo.sh          # or ./demo.sh 2019 for one year, month by month
```

### 10. event-driven-systems-in-go

**What it shows:** event-driven architecture in a car plant's yard. Three Go services (final inspection, parking management, parking) never call each other; they publish and react to events through RabbitMQ, including a buffer zone for when the lot is full. The demo plays the factory floor in four phases, slowly enough to read the logs.

**Needs:** Docker with Compose. Go is not needed.

**Branches:** `main`

```bash
git clone event-driven-systems-in-go.bundle
cd event-driven-systems-in-go
./demo.sh          # about 3.5 minutes
```

### 11. resilience-patterns-in-go

**What it shows:** two Go services, orders and inventory, used as a test bed. Inventory can be made slow or flaky while it runs, and each branch adds one resilience pattern to the orders service, shown failing first and then fixed. Each branch builds on the one before it. Standard library only.

**Needs:** Go.

**Branches:** `main` (the baseline, with no patterns), `01-timeout-pattern`, `02-retry-pattern`, `03-circuit-breaker`, `04-bulk-head`, `05-rate-limiter`, `06-fallback`, `07-hedged-requests`, `08-deadline-propagation`, `09-load-shedding`, `10-retry-budget`, `11-adaptive-concurrency`

```bash
git clone resilience-patterns-in-go.bundle
cd resilience-patterns-in-go
./demo.sh
git switch 01-timeout-pattern
./demo.sh
```

### 12. k8s-otel-observability

**What it shows:** OpenTelemetry on Kubernetes with no application code. A shop of four nginx services sends traces, metrics and logs to the OpenTelemetry Collector, which stores them in Tempo, Prometheus and Loki; Grafana shows all three, with each request linked to its trace. Every container is an official image.

**Needs:** Docker Desktop with Kubernetes turned on. The first run downloads about 1.5 GB of images.

**Branches:** `main`

```bash
git clone k8s-otel-observability.bundle
cd k8s-otel-observability
./demo.sh          # when it says Ready, open http://localhost:3000
./demo.sh down
```
