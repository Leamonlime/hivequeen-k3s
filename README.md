# Home Kubernetes Server (hivequeen)

Single-node, self-hosted Kubernetes cluster running on repurposed hardware — the
infrastructure that hosts [Ghost](https://github.com/Leamonlime/ghost), a personal
AI assistant, along with its databases and supporting services. Built as a real,
load-bearing home server rather than a throwaway lab cluster.

## Stack

| Layer | Choice | Notes |
| --- | --- | --- |
| OS | Ubuntu Server 24.04.4 LTS | Single node, `hivequeen` |
| Orchestration | k3s v1.35.5 | Lightweight Kubernetes, single-node control plane |
| Ingress | Traefik | Bundled with k3s, `LoadBalancer` on the LAN |
| Databases | PostgreSQL + Redis | Deployed via Bitnami Helm charts |
| Local LLM | Ollama (`llama3.2`) | Runs on the host for privacy-sensitive workloads |
| Management | Portainer | Visual container/cluster management |
| File sync | Syncthing | Private peer-to-peer sync, no cloud dependency |
| Storage | `local-path-provisioner` | k3s default, node-local persistent volumes |

## Architecture

- **Apps run as pods** with their configuration and credentials injected from
  Kubernetes `Secret`s via `secretKeyRef` — **zero inline secret values in any
  manifest** (see [`manifests/`](manifests/)).
- **Stateful services** (Postgres, Redis) are managed with Helm so upgrades and
  chart values stay declarative and reproducible (see
  [`manifests/helm-values.example.yaml`](manifests/helm-values.example.yaml)).
- **Namespacing** keeps each application's workloads isolated.

## Networking

- **Tailscale** mesh network for secure remote access — the server is never
  exposed to the public internet; no port-forwarding, no public ingress.
- Reserved/static LAN addressing, replacing the initial DHCP setup.

## Reliability

- Automatic filesystem checks on boot.
- **Planned:** UPS-backed graceful shutdown on power loss via NUT (Network UPS
  Tools) — hardware acquired, not yet wired in.

## Why self-hosted

Full control over personal and financial data instead of handing it to
third-party cloud infrastructure, and a hands-on way to learn Kubernetes on a
real system I actually depend on daily rather than a toy cluster.

## Repository layout

```
.
├── README.md
├── .env.example                         # every env var / secret key, placeholder values only
├── docs/
│   └── setup.md                         # how the cluster was built, step by step
└── manifests/
    ├── namespace.example.yaml           # per-app namespace
    ├── app-secret.example.yaml          # Secret shape (placeholders, never real values)
    ├── app-deployment.example.yaml      # Deployment + Service using secretKeyRef
    └── helm-values.example.yaml         # sanitized Postgres/Redis Helm values
```

Everything in this repo is **sanitized**: manifests are generic templates, and no
real credentials, tokens, kubeconfigs, or cluster certificates are committed
anywhere in the history.

## Status

Live, core services running since mid-2026. Hardware and reliability upgrades
(UPS/NUT, faster storage) planned.

## License

[MIT](LICENSE)
