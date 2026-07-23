Presentation Script

## Slide 1 — Introduction

Hello, my name is Dale Murphy, and this is my DevOps with AWS final project.

For this project, I used the Saleor e-commerce platform. My goal was to take a real multi-service application, run it locally with Docker, build automated deployment pipelines, publish the container image, and deploy the application to AWS.

---

## Slide 2 — What I Built

The main application is Saleor.

Saleor includes a Python and Django backend, a GraphQL API, an administrative dashboard, a PostgreSQL database, and Redis.

I also used Mailpit for testing email and Jaeger for tracing.

Instead of installing every part directly on my computer, I placed the services in containers.

A container packages the application and the software it needs so that it can run more consistently in different environments.

---

## Slide 3 — Docker

I created a Dockerfile for the Saleor API.

The Dockerfile begins with a Python base image. It installs the required system packages and Python dependencies, copies the application code, and defines the command that starts the application.

I also configured the Saleor Dashboard. Because the local Dashboard build required too much memory on my Intel Mac, I used an official prebuilt Dashboard image. This was a practical solution that allowed the full environment to run successfully.

---

## Slide 4 — Docker Compose

Saleor needs more than one container, so I used Docker Compose.

Docker Compose starts and connects all of the services from one YAML file.

My Compose environment included:

- Saleor API
- Saleor Dashboard
- PostgreSQL
- Redis
- Mailpit
- Jaeger

I used health checks so Docker could determine whether important services were ready.

I verified the Compose file with the command:

```bash
docker compose config
```

Then I started everything with:

```bash
docker compose up -d
```

---

## Slide 5 — Local Testing

After the containers started, I tested each local service.

The API ran on port 8000.

The Dashboard ran on port 9000.

Mailpit ran on port 8025.

Jaeger ran on port 16686.

This step was important because it proved the application worked locally before I attempted to deploy it to AWS.

---

## Slide 6 — GitHub Branches

I used three GitHub branches for the deployment process:

- develop
- staging
- production

Each branch represents a different stage of the software-delivery process.

The develop branch is used for early changes.

The staging branch is used for testing changes before release.

The production branch represents the version intended for deployment.

---

## Slide 7 — GitHub Actions

I created a GitHub Actions workflow for each branch.

When I push code, GitHub Actions automatically performs several jobs.

It checks out the repository, installs tools, runs code-quality checks, audits dependencies, builds the Docker image, scans the image, tests the application, and pushes the image to Docker Hub.

This process is called continuous integration and continuous delivery, or CI/CD.

It reduces manual work and makes the delivery process repeatable.

---

## Slide 8 — Security Checks

I included multiple automated checks.

Ruff checks the Python code for quality and formatting problems.

pip-audit checks Python packages for known vulnerabilities.

Trivy scans the finished Docker image for operating-system and library vulnerabilities.

These checks help find problems before an image is deployed.

---

## Slide 9 — Docker Hub

After GitHub Actions builds and tests an image, it pushes the image to Docker Hub.

My Docker Hub repository contains three tags:

- develop
- staging
- production

A tag identifies which version or environment the image belongs to.

AWS pulls the production image from Docker Hub when it starts the ECS task.

---

## Slide 10 — AWS ECS and Fargate

For cloud deployment, I used Amazon ECS.

ECS stands for Elastic Container Service. It manages and runs containers in AWS.

I selected Fargate as the launch type.

Fargate allows AWS to manage the underlying servers. I provide the container configuration, CPU, and memory requirements, and AWS runs the containers.

I created an ECS cluster named `saleor-production-cluster`.

---

## Slide 11 — Task Definition

I created an ECS task definition named `saleor-production`.

A task definition is similar to a blueprint for the application.

It tells ECS:

- Which container images to use
- Which ports to expose
- How much CPU and memory are needed
- Which environment variables are required
- Where logs should be sent

The task definition included the Saleor API and its supporting services.

---

## Slide 12 — ECS Service

I created an ECS service from the task definition.

The service is responsible for keeping the requested number of tasks running.

If a task stops unexpectedly, ECS can attempt to replace it.

The service also connects the application to the load balancer.

---

## Slide 13 — Application Load Balancer

I created an Application Load Balancer.

The load balancer receives incoming HTTP requests and sends them to the Saleor API container.

The load balancer uses a target group and health checks.

A health check periodically requests the application endpoint. If the application responds successfully, the target is considered healthy.

---

## Slide 14 — Problem Solving

The deployment did not work perfectly on the first attempt.

One major error was:

```text
relation "django_site" does not exist
```

This meant the Saleor application was running, but the database had not been initialized.

Saleor is a Django application, and Django uses migrations to create its database tables.

I updated the API startup command so that it runs:

```bash
python manage.py migrate --noinput
```

before starting Gunicorn.

The startup process also waits and retries if PostgreSQL is not ready yet.

This solved the missing-table problem.

---

## Slide 15 — Other Challenges

I also solved several other problems during the project:

- YAML indentation errors
- Invalid Docker image tags
- Incorrect GitHub secret names
- Docker memory limitations
- Docker disk-space problems
- Health-check timing problems
- AWS role configuration
- Load-balancer target failures

I learned to troubleshoot by reading the exact error message, checking logs, validating configuration files, and changing one item at a time.

---

## Slide 16 — Architecture

The deployment flow is:

```text
GitHub
   │
   ▼
GitHub Actions
   │
   ▼
Docker Hub
   │
   ▼
Amazon ECS with Fargate
   │
   ▼
Application Load Balancer
   │
   ▼
Saleor Application
```

I push the code to GitHub.

GitHub Actions builds and tests the Docker image.

The image is stored in Docker Hub.

Amazon ECS pulls and runs the production image.

The Application Load Balancer sends user traffic to the running Saleor application.

---

## Slide 17 — Final Result

At the end of the project, I had:

- A working local Docker Compose environment
- Separate develop, staging, and production pipelines
- Successful GitHub Actions runs
- Docker images stored in Docker Hub
- An ECS cluster
- An ECS task definition
- A running ECS task
- An ECS service
- An Application Load Balancer
- A successful AWS deployment

---

## Slide 18 — What I Learned

This project helped me understand how different DevOps tools work together.

Docker packages the application.

Docker Compose manages the local multi-container environment.

GitHub Actions automates building and testing.

Docker Hub stores the images.

Amazon ECS and Fargate run the production containers.

The Application Load Balancer routes traffic.

CloudWatch provides logs for troubleshooting.

Most importantly, I learned that DevOps is not only about writing configuration files. It is also about testing, reading logs, understanding failures, and improving the deployment until it works reliably.

Thank you.
