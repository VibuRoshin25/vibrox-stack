# vibrox-stack

`vibrox-stack` is the orchestration and deployment layer for the **Vibrox** microservices suite.  
It includes Docker Compose, Kubernetes manifests, and Git submodules to manage and run all core services in one place.

---

## 📦 Included Services

| Service        | Repo                                                        | Description                               |
| -------------- | ----------------------------------------------------------- | ----------------------------------------- |
| `vibrox-core`  | [GitHub Link](https://github.com/VibuRoshin25/vibrox-core)  | Systems Lab API gateway (Go, REST + gRPC) |
| `vibrox-echo`  | [GitHub Link](https://github.com/VibuRoshin25/vibrox-echo)  | Centralized logging (Go, gRPC)            |
| `vibrox-dns`   | [GitHub Link](https://github.com/VibuRoshin25/vibrox-dns)   | Forwarding DNS (Go, UDP + TCP)            |
| `vibrox-web`   | [GitHub Link](https://github.com/VibuRoshin25/vibrox-web)   | Portfolio and live systems lab            |
| `vibrox-arena` | [GitHub Link](https://github.com/VibuRoshin25/vibrox-arena) | Tic-tac-toe bot and gRPC experiment       |

`vibrox-auth` is retained as a historical learning repository but is no longer
part of the runtime. Its former user-management flow had no product requirement.
`vibrox-arena` provides an interactive, inspectable gRPC demonstration through
the tic-tac-toe bot.

---

## 🚀 Getting Started

### 🔁 Clone with Submodules

```bash
git clone --recurse-submodules https://github.com/VibuRoshin25/vibrox-stack.git
cd vibrox-stack
```

> Already cloned? Run this to pull submodules:

```bash
git submodule update --init --recursive
```

---

## 🐳 Run with Docker Compose

You can spin up all services locally using Docker Compose:

```bash
docker-compose up --build
```

Test the local DNS service with `dig @127.0.0.1 -p 2053 example.com`. It is
published on loopback only; set `DNS_UPSTREAM` to select another upstream
resolver.

Open `http://localhost:3000` for the portfolio and interactive Systems Lab.
Requests under `/api` are routed internally to `vibrox-core` by the web
container, keeping backend services behind a same-origin entry point.

All Compose services use `vibrox-dns` as their default resolver. The DNS
service forwards through Docker's embedded resolver, so Compose service names
such as `logger` and `vibrox-core` continue to resolve normally.

> Make sure each submodule has its own `Dockerfile` and `.env`.

---

## ☸️ Run with Kubernetes

If you're using Kubernetes:

1. Navigate to the `manifests/` directory.
2. Apply the manifests:

```bash
kubectl apply -f manifests/
```

> Customize the manifests for your local cluster or cloud setup.
