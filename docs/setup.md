# Setup notes — hivequeen k3s

How the cluster was built, in order. Commands are illustrative; adapt paths and
names to your environment. **No secrets or real tokens appear here.**

## 1. Base OS

- Ubuntu Server 24.04.4 LTS on a single repurposed machine (`hivequeen`).
- Static/reserved LAN address instead of DHCP so service endpoints are stable.

## 2. Install k3s (single-node control plane)

```bash
curl -sfL https://get.k3s.io | sh -
```

k3s ships with Traefik (ingress) and `local-path-provisioner` (storage) out of the
box, which is why this cluster needs no separate ingress controller or CSI driver.

The admin kubeconfig lands at `/etc/rancher/k3s/k3s.yaml`. **Never commit it.** To
use `kubectl` from another machine, copy it out and edit the `server:` field to the
node's Tailscale/LAN address:

```bash
sudo cat /etc/rancher/k3s/k3s.yaml            # copy to ~/.kube/hivequeen.yaml locally
export KUBECONFIG=~/.kube/hivequeen.yaml
```

## 3. Remote access with Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up            # authenticate via the printed URL (or a TAILSCALE_AUTHKEY)
```

The server stays entirely off the public internet — access is over the tailnet
only. No port-forwarding, no public ingress.

## 4. Helm + stateful services

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

kubectl create namespace myapp
helm install myapp-postgres bitnami/postgresql -n myapp -f manifests/helm-values.example.yaml
helm install myapp-redis    bitnami/redis      -n myapp -f manifests/helm-values.example.yaml
```

Each chart generates its own password Secret (`myapp-postgresql`, `myapp-redis`);
apps read those via `secretKeyRef` rather than hard-coding anything.

## 5. Application workloads

Images are built locally and imported into containerd (no external registry):

```bash
docker build -t myapp:latest .
docker save myapp:latest | sudo k3s ctr images import -
```

Then apply the manifests (secret first, so the deployment's `secretKeyRef`s resolve):

```bash
kubectl apply -f manifests/namespace.example.yaml
kubectl -n myapp create secret generic myapp-secrets --from-literal=...   # out-of-band
kubectl apply -f manifests/app-deployment.example.yaml
```

## 6. Local LLM (Ollama)

Runs directly on the host (not in k8s) so privacy-sensitive inference never leaves
the machine:

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2
```

Pods reach it at `http://<host-ip>:11434` on the cluster network.

## 7. Management UI (Portainer)

```bash
helm repo add portainer https://portainer.github.io/k8s/
helm install portainer portainer/portainer -n portainer --create-namespace
```

The admin account is created on first visit through the web UI — no credential is
stored in git.

## 8. Reliability

- Automatic filesystem checks on boot.
- **Planned:** UPS + NUT (Network UPS Tools) for graceful shutdown on power loss.
