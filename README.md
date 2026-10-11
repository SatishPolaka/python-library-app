# Python Application CI/CD Pipeline Using Jenkins and Docker Swarm

Automated CI/CD pipeline for a Python-based application using Jenkins, SonarQube, Docker, Trivy, Docker Hub, and Docker Swarm.

## Overview

This project demonstrates an end-to-end CI/CD workflow that automates source code checkout, code quality analysis, Docker image creation, vulnerability scanning, image publishing, and application deployment.

Jenkins orchestrates the pipeline, while Docker Swarm manages the application services across a cluster consisting of one manager node and two worker nodes.

The objective is to make application delivery consistent, repeatable, and easier to maintain.

## Architecture

```text
                    ┌──────────────────┐
                    │      GitHub      │
                    │ Application Code │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      Jenkins     │
                    │   CI/CD Pipeline │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │    SonarQube     │
                    │  Code Analysis   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   Quality Gate   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   Docker Build   │
                    │  Five Images     │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      Trivy       │
                    │ Image Scanning   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │    Docker Hub    │
                    │ Image Registry   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   Docker Stack   │
                    │    Deployment    │
                    └────────┬─────────┘
                             │
                             ▼
              ┌────────────────────────────┐
              │     Docker Swarm Cluster   │
              │                            │
              │  Manager + 2 Worker Nodes  │
              │                            │
              │  Database                  │
              │  Authentication Service    │
              │  Book Service              │
              │  Borrow Service            │
              │  Frontend Application      │
              └────────────────────────────┘
```

## Technology Stack

| Technology   | Purpose                                           |
| ------------ | ------------------------------------------------- |
| AWS EC2      | Infrastructure for Jenkins and Docker Swarm       |
| Jenkins      | CI/CD pipeline automation                         |
| Git          | Source code version control                       |
| GitHub       | Source code repository                            |
| SonarQube    | Static code quality analysis                      |
| Docker       | Container image creation and execution            |
| Trivy        | Container image vulnerability and secret scanning |
| Docker Hub   | Container image registry                          |
| Docker Swarm | Container orchestration                           |
| Docker Stack | Multi-service application deployment              |

## Source Code

**GitHub Repository:**
https://github.com/SatishPolaka/python-library-app

The repository contains the application source code, individual service Dockerfiles, database configuration, and Docker Compose configuration used for deployment.

## Application Components

The application is divided into five containerised components.

| Component              | Docker Image     | Docker Hub Image                 |
| ---------------------- | ---------------- | -------------------------------- |
| Authentication service | `auth-service`   | `satishpolaka/auth-service:v1`   |
| Book service           | `book-service`   | `satishpolaka/book-service:v1`   |
| Borrow service         | `borrow-service` | `satishpolaka/borrow-service:v1` |
| Database               | `dbimage`        | `satishpolaka/dbimage:v1`        |
| Frontend application   | `python-app`     | `satishpolaka/python-app:v1`     |

Each image is built from its corresponding Dockerfile and published to Docker Hub for use during deployment.

## Jenkins CI/CD Pipeline

The Jenkins pipeline contains the following stages.

### 1. CODE — Source Code Checkout

Jenkins retrieves the application source code from GitHub and checks it out into the Jenkins workspace.

This workspace is used by the subsequent pipeline stages.

### 2. SCQ — SonarQube Analysis

The SonarQube Scanner analyses the application source code.

Project configuration:

* Project key: `python-library-app`
* Source directory: Jenkins workspace

The analysis results are uploaded to SonarQube for review.

### 3. QUALITY GATES

Jenkins waits for the SonarQube Quality Gate result.

The pipeline is configured to abort if the Quality Gate fails. This prevents subsequent stages from executing when the configured quality criteria are not satisfied.

### 4. IMAGES — Docker Image Build

Jenkins builds five Docker images using their respective Dockerfiles.

```text
auth/       → auth-service
book/       → book-service
borrow/     → borrow-service
database/   → dbimage
./          → python-app
```

The images are then available locally for security scanning and publishing.

### 5. IMAGE SCAN — Trivy

Trivy scans each locally built image.

The pipeline creates separate text reports:

| Image          | Report                         |
| -------------- | ------------------------------ |
| Authentication | `auth-image-trivyreport.txt`   |
| Book           | `book-image-trivyreport.txt`   |
| Borrow         | `borrow-image-trivyreport.txt` |
| Database       | `db-image-trivyreport.txt`     |
| Frontend       | `python-app-trivyreport.txt`   |

The reports contain the scanner output and can be reviewed to investigate identified vulnerabilities.

**Note:** The current pipeline records Trivy findings but does not automatically fail the build based on vulnerability severity.

### 6. REGISTRY — Docker Hub Publishing

Jenkins authenticates to Docker Hub using credentials configured in Jenkins.

Each image is tagged with the Docker Hub repository name and version `v1`, then pushed to the registry.

Images published:

```text
satishpolaka/auth-service:v1
satishpolaka/book-service:v1
satishpolaka/borrow-service:v1
satishpolaka/dbimage:v1
satishpolaka/python-app:v1
```

These images are referenced by the Docker Stack deployment configuration.

### 7. DEPLOY — Docker Stack

The final stage deploys the application using the Compose file:

```bash
docker stack deploy myapp --compose-file=compose.yml
```

The stack name is `myapp`.

Docker Swarm manages the services according to the deployment configuration, including replica counts, published ports, networking, volumes, and resource limits.

## Docker Swarm Architecture

The cluster consists of:

| Node           | Role    | Responsibility                            |
| -------------- | ------- | ----------------------------------------- |
| Swarm Manager  | Manager | Cluster management and service scheduling |
| Swarm Worker 1 | Worker  | Runs scheduled application tasks          |
| Swarm Worker 2 | Worker  | Runs scheduled application tasks          |

The manager maintains the cluster's desired state and coordinates service scheduling. Worker nodes execute the tasks assigned by the Swarm manager.

Services communicate over an overlay network defined in the Compose file.

## Deployment Configuration

The `compose.yml` file defines:

* Five application services.
* Docker Hub image references.
* Published ports.
* Service replica counts.
* An overlay network.
* Named volumes.
* CPU and memory resource limits and reservations.

### Published Ports

The configuration uses the following port mappings:

| Service        | Published Port | Container Port |
| -------------- | -------------: | -------------: |
| Frontend       |           5000 |           5000 |
| Authentication |           5001 |           5001 |
| Book           |           5002 |           5002 |
| Borrow         |           5003 |           5003 |
| Database       |           3306 |           3306 |

Actual availability depends on the application listening ports, network configuration, and EC2 security group rules.

### Persistent Storage

A named volume is configured for the database at `/var/lib/mysql`.

Named volumes allow database files to persist beyond the lifecycle of an individual container on the same Docker host.

**Important:** Default local Docker volumes are node-specific. They do not automatically replicate data between Swarm nodes. Production deployments requiring database availability across nodes need an appropriate shared storage or database replication strategy.

## Validation and Troubleshooting

After deployment, the following commands can be used on the Swarm manager to inspect the cluster.

**Check Swarm nodes**

```bash
docker node ls
```

**List deployed stacks**

```bash
docker stack ls
```

**Check application services**

```bash
docker stack services myapp
```

**Inspect service tasks**

```bash
docker stack ps myapp
```

**Check containers running on the current node**

```bash
docker ps
```

**Inspect a service's logs**

```bash
docker service logs myapp_auth_service
```

Service names should be verified using `docker stack services myapp` if they differ from the expected names.

## Security Considerations

The following practices are important when extending this project for production use:

* Store Docker Hub credentials in Jenkins Credentials rather than hardcoding them.
* Use least-privilege access for Jenkins and infrastructure accounts.
* Avoid exposing the Docker daemon socket to untrusted users.
* Review and remediate Trivy findings before releasing images.
* Avoid storing database passwords or other secrets in public repositories.
* Restrict database access to the required application services.
* Use TLS for external application traffic.
* Monitor application health, resource utilisation, and deployment status.
* Plan for persistent storage and database recovery.
* Use controlled image versions and a documented release process.

## Current Limitations and Production Considerations

This project demonstrates the core CI/CD and container orchestration workflow. Additional work would be required before treating it as a production-ready deployment.

Potential improvements include:

* Enforcing Trivy vulnerability thresholds.
* Adding application health checks and readiness handling.
* Using a secure secret-management solution.
* Implementing a persistent storage strategy suitable for rescheduling.
* Configuring monitoring, alerting, and centralised logging.
* Adding automated rollback and release approval processes.
* Testing database backup and recovery.
* Reviewing resource limits against actual application usage.

## Conclusion

This project demonstrates an automated CI/CD workflow that connects source control, code quality analysis, container image building, security scanning, image publishing, and container orchestration.

Jenkins coordinates the pipeline, SonarQube evaluates source code quality, Trivy scans container images, Docker Hub stores the images, and Docker Swarm manages application deployment.

The project provides practical experience in integrating DevOps tools into a repeatable application delivery workflow.


-------------------------------------------------------------------------------------------------
Clone the repo 

docker build -t dbimage database/ <br />
docker build -t authimage auth/ <br />
docker build -t bookimage book/ <br />
docker build -t borrowimage borrow/ <br />
docker build -t appimage . <br /> <br /> <br /> <br />


docker network create mynet <br /> <br /> <br /> <br />


docker run -d --name db -p 3306:3306 --network mynet dbimage <br />
docker run -d --name auth_service --network mynet  -p 5001:5001 authimage <br />
docker run -d --name book_service -p 5002:5002 --network mynet bookimage <br />
docker run -d --name borrow_service  -p 5003:5003 --network mynet borrowimage <br />
docker run -d --name frontend  -p 5000:5000 --network mynet  appimage <br /> 

 
