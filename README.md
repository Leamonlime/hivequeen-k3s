# Home Kubernetes Server (hivequeen)

Single-node self-hosted Kubernetes cluster running on repurposed hardware,
hosting a personal AI assistant, its databases, and supporting services.

## Stack
- k3s v1.35.5 on Ubuntu Server 24.04.4 LTS
- Portainer for visual container/cluster management
- Postgres, Redis, Ollama (llama3.2) for privacy-sensitive workloads
- Syncthing for private file sync, no cloud dependency

## Networking
- Tailscale mesh network for secure remote access, no public exposure required
- Reserved/static addressing in progress, replacing initial DHCP setup

## Reliability
- UPS for graceful shutdown on power loss, via NUT (Network UPS Tools)
- Automatic filesystem checks on boot

## Why self-hosted
Full control over personal and financial data instead of third-party cloud
infrastructure, and a hands-on way to learn Kubernetes on a real, load-bearing
system rather than a toy cluster.

## Status
Live, core services running since mid-2026. Hardware upgrades planned.
