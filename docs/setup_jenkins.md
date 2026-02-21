# Jenkins Setup Guide

A guide for running Jenkins with Docker, including Docker-in-Docker support, agent configuration, and pipeline basics.

---

## Jenkins with Docker Setup

### Dockerfile

```dockerfile
FROM jenkins/jenkins:lts-jdk21

USER root

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install prerequisites
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    lsb-release \
    python3-pip \
    git \
    curl \
    maven \
    ansible \
    ca-certificates \
    gnupg-agent \
    && rm -rf /var/lib/apt/lists/*

# Add Docker's official GPG key and repository
RUN curl -fsSL https://download.docker.com/linux/debian/gpg \
    | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
    https://download.docker.com/linux/debian $(lsb_release -cs) stable" \
    > /etc/apt/sources.list.d/docker.list

# Install Docker CLI
RUN apt-get update && \
    apt-get install -y --no-install-recommends docker-ce-cli && \
    rm -rf /var/lib/apt/lists/*

# Handle docker group for socket access
RUN if getent group docker; then \
      usermod -aG docker jenkins; \
    else \
      groupadd -g 989 docker && usermod -aG docker jenkins; \
    fi

USER jenkins

# Install Jenkins plugins
RUN jenkins-plugin-cli --plugins \
    blueocean \
    docker-workflow \
    git \
    job-dsl \
    configuration-as-code \
    credentials-binding \
    workflow-aggregator \
    workflow-cps \
    dark-theme \
    locale
```

---

### docker-compose.yml

```yaml
services:
  jenkins:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      TZ: "Europe/Warsaw"
    networks:
      - jenkins
    restart: unless-stopped

networks:
  jenkins:
    driver: bridge
```

### Build and Start

```bash
docker compose up -d --build
```

### Verify

```bash
# Check Jenkins is running
docker logs jenkins

# Verify Docker access from Jenkins container
docker exec -it jenkins docker ps
```

Access Jenkins at `http://server.example.com:8080`

---

## Jenkins Plugins

Essential plugins installed via `jenkins-plugin-cli`:

| Plugin | Purpose |
|---|---|
| `blueocean` | Modern pipeline UI |
| `docker-workflow` | Docker pipeline support |
| `git` | Git SCM integration |
| `job-dsl` | Programmatic job creation |
| `configuration-as-code` | Jenkins config as YAML |
| `credentials-binding` | Secure credential injection |
| `workflow-aggregator` | Pipeline aggregation |

---

## Build Configuration

### Discard Old Builds

Keep storage under control by configuring build retention:

- **Days to keep builds:** e.g., 30
- **Max # of builds to keep:** e.g., 10

### Build Triggers

| Trigger | Description |
|---|---|
| GitHub hook trigger | Builds on push via webhook |
| Poll SCM | Periodically checks for changes (`H/5 * * * *`) |
| Build periodically | Cron-based schedule |
| Build after other projects | Chain builds together |
| Trigger builds remotely | Via authentication token |

### Parameterized Builds

Enable `This project is parameterized` to add:
- String parameters
- Choice parameters
- Boolean parameters
- File parameters

---

## Jenkinsfile (Declarative Pipeline)

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myapp'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/your_username/your-repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm ${DOCKER_IMAGE}:${DOCKER_TAG} npm test'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose up -d'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```

---

## Jenkins Agent with Docker

### Agent docker-compose.yml

```yaml
services:
  jenkins-agent:
    image: jenkins/inbound-agent:latest
    container_name: jenkins-agent
    environment:
      JENKINS_URL: "http://jenkins:8080"
      JENKINS_SECRET: "your_api_token_here"
      JENKINS_AGENT_NAME: "docker-agent"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - jenkins

networks:
  jenkins:
    external: true
```

### Configure Agent in Jenkins UI

1. Go to `Manage Jenkins > Nodes > New Node`
2. Set node name and select `Permanent Agent`
3. Configure:
   - Remote root directory: `/home/jenkins`
   - Labels: `docker`
   - Launch method: `Launch agent by connecting it to the controller`
4. Copy the secret token to agent configuration

---

## Cache Management

### Workspace Cleanup

Install the **Workspace Cleanup Plugin**, then configure:

- **Delete workspace before build starts** - clean slate for each build
- **Delete workspace when build is done** - free space after completion

### Pipeline Cleanup

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'echo "Building..."'
            }
        }
    }
    post {
        always {
            // Clean workspace
            deleteDir()

            // Clean Docker cache
            sh 'docker system prune -af --volumes || true'
        }
    }
}
```

### Docker Prune (Manual)

```bash
# Remove all unused containers, images, networks, volumes
docker system prune -af --volumes

# Remove only dangling images
docker image prune -f
```

---

## Troubleshooting

### JENKINS_HOME

Jenkins stores all data in `/var/jenkins_home`. If using Docker volumes, ensure proper permissions:

```bash
# Fix ownership issues
sudo chown -R 1000:1000 ./jenkins_home
```

### Docker Socket Access

If Jenkins cannot access Docker:

```bash
# Check socket permissions
ls -la /var/run/docker.sock

# Add jenkins user to docker group inside container
docker exec -u root jenkins usermod -aG docker jenkins
docker restart jenkins
```

### View Logs

```bash
docker logs jenkins -f --tail 100
```
