# Master Roadmap & Audit Preparation Guide

This document breaks down every single requirement, implementation step, and audit question needed to successfully complete the Docker Microservices project based on the instructions and the grading rubric you provided.

---

## 0. Current Project Status & VM Strategy

To directly answer your project execution questions:

### Will the old VMs work?
**NO.** The old project used Vagrant to run 3 separate, heavy VirtualBox VMs. This project explicitly requires Docker.
- **Action Taken:** The old VMs have been successfully destroyed (`vagrant destroy -f`).
- **Required Setup:** You now only need **ONE** single Linux virtual machine. You will install Docker and Docker Compose on this machine, and it will run all 6 isolated "containers" internally.

### What has been achieved vs what is remaining?
- **Achieved:** We have the application logic (the Python/Node code inside `srcs/`), the Postman tests, and this Master Roadmap outlining the strict grading rules.
- **Remaining:** 100% of the Docker orchestration. We have not written any Dockerfiles, we have no `docker-compose.yml`, and your backend codebase still connects to old static IP addresses.

### What TO DO vs What NOT TO DO (Instant Audit Fails)
- **DO** use exactly the names `inventory-database`, `billing-database`, `RabbitMQ`.
- **DO NOT** use `latest` version tags.
- **DO NOT** use prebuilt database images (e.g., `FROM postgres`). Write out the installation via `apt-get` on a `debian:bullseye` base!
- **DO NOT** expose any `ports:` for your databases in docker-compose. Only `api-gateway-app` sits on port 3000.
- **DO NOT** commit your `.env` password file to Git.

---

## 1. Core Architecture & Strict Requirements Checklist

Before writing any code, you must ensure your setup strictly follows these rules as they are explicit grading criteria:

- [ ] **No `latest` tags**: You must use a specific version tag.
- [ ] **Penultimate OS Base**: Every `Dockerfile` must use the "penultimate" (stable minus one) version of Debian (e.g., `bullseye` if `bookworm` is latest) or Alpine (e.g., `3.19` if `3.20` is latest). 
- [ ] **Custom Builds Only**: You cannot use `postgres:14` or `rabbitmq:3` directly. You must use `FROM debian:bullseye` and run `apt-get install` to install Postgres, RabbitMQ, Node.js/Python, etc.
- [ ] **Minimize Image Layers**: Combine your `RUN` commands (e.g., `RUN apt-get update && apt-get install -y ... && rm -rf /var/lib/apt/lists/*`) to avoid useless layers and keep image sizes small.
- [ ] **Strict Naming Match**: The service name in `docker-compose.yml` MUST exactly match the Docker Image name, and MUST match the names expected by the audit script: `inventory-database`, `billing-database`, `inventory-app`, `billing-app`, `RabbitMQ`, `api-gateway-app`.
- [ ] **Restart Policy**: Every container must have `restart: on-failure` in the compose file to satisfy the `docker inspect` check.
- [ ] **Secure Credentials**: ALL passwords must be in an `.env` file, and `.env` MUST be in `.gitignore`. No hardcoded credentials in Dockerfiles or compose configs.
- [ ] **Network Exposure Restrictions**: Only `api-gateway-app` should map ports to the host (`3000:3000`). Databases and backend apps must only be reachable internally.
- [ ] **Required Volumes**: You must create specifically named volumes: `inventory-database`, `billing-database`, and `api-gateway-app`.

---

## 2. Step-by-Step Implementation Steps

### Step 1: Repository Setup
1. Clone your project structure.
2. Create an `.env` file. Put all Postgres user/passwords, RabbitMQ user/pass here.
3. Check your `.gitignore` to ensure `.env` is listed. 

### Step 2: Create the 6 Dockerfiles
You need 6 directories, each with a `Dockerfile`.
**Example structure for a Database (`inventory-database/Dockerfile`):**
```dockerfile
FROM debian:bullseye 
# Do not use 'latest'
# Combine commands to save layers
RUN apt-get update && apt-get install -y postgresql && rm -rf /var/lib/apt/lists/*
... (rest of setup)
```
*(You will do this for `billing-database` and `RabbitMQ` as well).*

**Example structure for an App (`inventory-app/Dockerfile`):**
```dockerfile
FROM debian:bullseye
RUN apt-get update && apt-get install -y nodejs npm ... # (or python3/pip depending on your CRUD apps)
COPY . /app
WORKDIR /app
RUN npm install # or pip install
CMD ["npm", "start"]
```

### Step 3: Write `docker-compose.yml`
This file ties everything together. It must look something like this:
```yaml
version: '3.8'

services:
  inventory-database:
    build: ./inventory-database
    image: inventory-database # Name matches service explicitly
    container_name: inventory-database
    restart: on-failure
    volumes:
      - inventory-database:/var/lib/postgresql/data # Pointing to the named volume
    networks:
      - microservices-net
    env_file:
      - .env

  api-gateway-app:
    build: ./api-gateway-app
    image: api-gateway-app
    container_name: api-gateway-app
    restart: on-failure
    ports:
      - "3000:3000" # THE ONLY EXPOSED PORT
    volumes:
      - api-gateway-app:/var/logs/api-gateway
    networks:
      - microservices-net

  # ... (ADD THE OTHER 4 SERVICES FOLLOWING THIS PATTERN) ...

volumes:
  inventory-database:
  billing-database:
  api-gateway-app:

networks:
  microservices-net:
```

### Step 4: Refactor App Connection Strings
Modify the code inside your apps to connect using the new Docker hostnames.
- API Gateway connects to: `http://inventory-app:8080`
- Billing App connects to RabbitMQ at: `amqp://user:pass@RabbitMQ:5672`
- Apps connect to Postgres databases using `inventory-database` and `billing-database`.

### Step 5: README Documentation
Write your `README.md` to cover:
- What the project does.
- Prerequisites (Docker installed).
- Setup instructions (`docker-compose up -d`).
- How to test endpoints.

---

## 3. Preparation for the Oral Audit Questions

The audit script requires you to verbally answer these questions. Here are the expected answers to memorize/understand:

**1. What are containers and what are their advantages?**
Containers are lightweight, standalone, executable packages that include everything needed to run a piece of software (code, runtime, libraries). **Advantages**: Fast to start, consistent across environments (works on my machine = works everywhere), isolated, and use fewer resources than VMs.

**2. What is the difference between containers and virtual machines?**
VMs virtualize the hardware and require a full, heavy guest Operating System for each VM. Containers virtualize the operating system level, sharing the host OS kernel, making them much faster and smaller.

**3. What is Docker and what is it used for?**
Docker is a platform that allows developers to easily create, deploy, and manage containerized applications. It standardizes the environment so code runs reliably anywhere.

**4. What is a microservices architecture and why use it?**
It is an architectural style that structures an application as a collection of loosely coupled, independently deployable services organized around business capabilities. **Why use it**: Easier scaling of specific parts, fault isolation (one crash doesn't take down the whole system), independent deployments, and flexibility in picking tech stacks for different services.

**5. What is a queue and what is it used for? What is RabbitMQ?**
A queue is a buffer that stores messages asynchronously until a receiving application processes them. It's used to decouple services, balance heavy loads, and provide resiliency. **RabbitMQ** is a popular open-source message broker software that implements this queuing mechanism.

**6. What is a Dockerfile and explain its instructions?**
A Dockerfile is a text document with commands to build a Docker image. Instructions:
- `FROM`: Sets the base image (e.g., Debian).
- `RUN`: Executes bash commands to install software (creates a layer).
- `COPY`: Copies files from your local host into the image.
- `WORKDIR`: Sets the working directory inside the container.
- `CMD`/`ENTRYPOINT`: The default command run when the container starts.

**7. What is a Docker volume and why use it?**
Volumes are mechanisms for persisting data generated by a container. **Why use it**: By default, data inside a container is lost when the container is deleted. A volume saves data safely on the host machine so your databases don't lose data on a restart.

**8. What is the Docker network and why use it?**
It is a virtual network created by Docker that allows isolated containers to communicate securely with each other using their container names as hostnames, without exposing them to the external internet.

**9. What is a Docker image and why use it?**
An image is a read-only template used to create containers. It contains the OS, software, and application code. **Why**: It serves as a portable snapshot of your app setup, ensuring reproducible builds anywhere.

---

## 4. Final Testing Checklist (The Audit Gauntlet)
Before presenting, run through exactly what the evaluator will do:

1. `docker-compose up -d` -> Does everything start with no errors?
2. `docker ps` -> Check if names are exactly `inventory-database`, `billing-database`, `RabbitMQ`, `inventory-app`, `billing-app`, `api-gateway-app`.
3. Check exposed ports: Only `api-gateway-app` should show `0.0.0.0:3000->3000/tcp`. No others.
4. `docker inspect -f "{{ .HostConfig.RestartPolicy }}" <container-name>` -> Must show `{on-failure 0}`.
5. `docker volume ls` -> Check for the 3 exact volume names.
6. Verify `.env` is ignored by running `git status` or checking `.gitignore`.
7. **Postman API Test 1**: POST `/api/movies` works (Status 200).
8. **Postman API Test 2**: GET `/api/movies` works (Status 200).
9. **The Resiliency Test**: 
   - POST `/api/billing` -> Expect 200.
   - Run `docker stop billing-app`
   - POST `/api/billing` again. **It must STILL return 200!** 
   - *(Explanation: Because the API Gateway dumps the request into the RabbitMQ queue successfully. The gateway doesn't wait for the billing app to finish. When you turn `billing-app` back on, it will fetch the queued messages!)*
