# UGC Ansible Deployment

Two deployment options are available:

## Option 1: Build on Server (Original)

Clones repos and builds Docker images directly on the server.

```bash
cd ansible
ansible-playbook -i inventory/production.ini playbooks/deploy.yml
```

**Pros**: Simple, no registry needed
**Cons**: Requires disk space and build time on server

---

## Option 2: Pre-built Images (Registry)

Build images locally, push to registry, pull on server.

### Step 1: Build and push images locally

```bash
# Build images for amd64 (server architecture)
./scripts/build-push.sh --push -u YOUR_DOCKERHUB_USER -t v1.0.0

# Or build without pushing first
./scripts/build-push.sh -u YOUR_DOCKERHUB_USER -t v1.0.0
docker push YOUR_DOCKERHUB_USER/ugc-ui:v1.0.0
docker push YOUR_DOCKERHUB_USER/ugc-admin:v1.0.0
```

### Step 2: Deploy to server

```bash
cd ansible
ansible-playbook -i inventory/production.ini playbooks/deploy-registry.yml \
  -e "registry_user=YOUR_DOCKERHUB_USER" \
  -e "tag=v1.0.0"

# With private registry authentication
ansible-playbook -i inventory/production.ini playbooks/deploy-registry.yml \
  -e "registry_user=YOUR_DOCKERHUB_USER" \
  -e "tag=v1.0.0" \
  -e "registry_password=YOUR_PASSWORD"
```

**Pros**: Fast deployment, no build on server, saves disk space
**Cons**: Requires registry account, need to build locally first

---

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `registry` | `docker.io` | Docker registry URL |
| `registry_user` | `ugc` | Registry username/namespace |
| `tag` | `latest` | Image tag |
| `branch` | `develop` | Git branch for API repos |
| `registry_password` | - | Registry password (for private registries) |

## Quick Commands

```bash
# Deploy with build on server
ansible-playbook -i inventory/production.ini playbooks/deploy.yml

# Deploy with pre-built images
ansible-playbook -i inventory/production.ini playbooks/deploy-registry.yml \
  -e "registry_user=myuser" -e "tag=v1.0.0"

# Deploy specific branch
ansible-playbook -i inventory/production.ini playbooks/deploy.yml \
  -e "branch=main"
```
