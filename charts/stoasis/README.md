# Stoasis Helm chart

Deploys the Stoasis frontend, Spring Boot backend, MCP service and a single-node
PostgreSQL database. The frontend and MCP endpoint are public; the backend and
database remain cluster-internal.

## Prerequisites

- Traefik ingress class `traefik-public`
- cert-manager `ClusterIssuer` named `letsencrypt-cloudflare`
- a default `StorageClass`
- a GHCR pull secret because the Stoasis images are private

Create the runtime secret before syncing the Argo CD application. Use a
URL-safe PostgreSQL password because it is also embedded in `database-url`:

```bash
POSTGRES_PASSWORD="replace-with-a-long-url-safe-password"
SESSION_ENCRYPTION_KEY="$(openssl rand -base64 32)"

sudo k3s kubectl create namespace stoasis --dry-run=client -o yaml | sudo k3s kubectl apply -f -
sudo k3s kubectl -n stoasis create secret generic stoasis-secrets \
  --from-literal=postgres-password="$POSTGRES_PASSWORD" \
  --from-literal=database-url="postgresql://stoasis:${POSTGRES_PASSWORD}@stoasis-postgresql:5432/stoasis" \
  --from-literal=session-encryption-key="$SESSION_ENCRYPTION_KEY" \
  --from-literal=resend-api-key="re_replace-me"
```

Create a classic GitHub PAT with `read:packages`, then create the registry
secret without committing the token to Git:

```bash
sudo k3s kubectl -n stoasis create secret docker-registry ghcr-stoasis \
  --docker-server=ghcr.io \
  --docker-username=GurkeSchaleSammler \
  --docker-password='YOUR_GITHUB_PAT'
```

GitOps defaults:

- Web app: `https://app.stoasis.ch`
- MCP: `https://mcp.stoasis.ch/mcp`
- RAM requests: 928 MiB total
- RAM limits: 1920 MiB total
- PostgreSQL PVC: 5 GiB
- Upload PVC: 10 GiB

