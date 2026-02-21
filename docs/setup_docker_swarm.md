# Docker Swarm Setup Guide

A guide for initializing and managing a Docker Swarm cluster for container orchestration across multiple nodes.

---

## Concepts

| Term | Description |
|---|---|
| **Manager** | Nodes that manage the cluster state and schedule services |
| **Worker** | Nodes that run containerized workloads |
| **Service** | A definition of tasks to run on the cluster |
| **Task** | A single container instance running as part of a service |
| **Overlay Network** | Multi-host networking for Swarm services |

> **Tip:** For high availability, use 3 or 5 manager nodes (always an odd number for Raft consensus).

---

## Initialize Swarm

### On the First Manager Node

```bash
docker swarm init --advertise-addr 10.0.0.10
```

This outputs join tokens for managers and workers.

### Retrieve Join Tokens

```bash
# Worker token
docker swarm join-token worker

# Manager token
docker swarm join-token manager
```

---

## Join Nodes

### Join as Worker

Run on each worker node:

```bash
docker swarm join \
  --token SWMTKN-1-xxxxxxxxxxxxx \
  10.0.0.10:2377
```

### Join as Manager

Run on additional manager nodes:

```bash
docker swarm join \
  --token SWMTKN-1-xxxxxxxxxxxxx \
  10.0.0.10:2377
```

### Verify Cluster

```bash
docker node ls
```

---

## Deploy Services

### Create a Service

```bash
docker service create \
  --name nginx \
  --replicas 3 \
  --publish 80:80 \
  nginx:latest
```

### List Services

```bash
docker service ls
docker service ps nginx
```

### Scale a Service

```bash
docker service scale nginx=5
```

### Update a Service

```bash
docker service update \
  --image nginx:1.25 \
  --update-parallelism 1 \
  --update-delay 10s \
  nginx
```

### Remove a Service

```bash
docker service rm nginx
```

---

## Stack Deployment (Compose Files)

### Deploy a Stack

```bash
docker stack deploy -c docker-compose.yml myapp
```

### Example Stack File

```yaml
# docker-compose.yml
services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
      placement:
        constraints:
          - node.role == worker
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"

  visualizer:
    image: dockersamples/visualizer:latest
    deploy:
      placement:
        constraints:
          - node.role == manager
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

### List Stacks

```bash
docker stack ls
docker stack ps myapp
docker stack services myapp
```

### Remove a Stack

```bash
docker stack rm myapp
```

---

## Multi-Manager HA

For production, run 3 manager nodes:

```bash
# On manager1 (first init)
docker swarm init --advertise-addr 10.0.0.10

# On manager2
docker swarm join --token <manager_token> 10.0.0.10:2377

# On manager3
docker swarm join --token <manager_token> 10.0.0.10:2377
```

### Promote / Demote Nodes

```bash
# Promote worker to manager
docker node promote <node_name>

# Demote manager to worker
docker node demote <node_name>
```

---

## Networking

### Create Overlay Network

```bash
docker network create \
  --driver overlay \
  --attachable \
  my-overlay
```

### Use in Service

```bash
docker service create \
  --name web \
  --network my-overlay \
  nginx:latest
```

---

## Monitoring and Troubleshooting

### Node Status

```bash
docker node ls
docker node inspect <node_name> --pretty
```

### Service Logs

```bash
docker service logs nginx -f --tail 100
```

### Drain a Node (Maintenance)

```bash
# Remove workloads from a node
docker node update --availability drain <node_name>

# Bring node back
docker node update --availability active <node_name>
```

### Leave Swarm

```bash
# On the node to remove
docker swarm leave

# Force (if manager)
docker swarm leave --force
```

### Required Ports

| Port | Protocol | Description |
|---|---|---|
| 2377 | TCP | Cluster management |
| 7946 | TCP/UDP | Node communication |
| 4789 | UDP | Overlay network traffic |
