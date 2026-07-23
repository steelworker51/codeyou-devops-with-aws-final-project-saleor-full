# Project Summary

I built and deployed a containerized version of the Saleor e-commerce platform using a complete DevOps workflow.

For local development, I used Docker and Docker Compose to run the Saleor API, Saleor Dashboard, PostgreSQL, Redis, Mailpit, and Jaeger. Docker Compose allowed all of the services to communicate through one shared network while keeping their configuration in a single YAML file.

I created separate GitHub Actions pipelines for the develop, staging, and production branches. Each pipeline checks the code, audits dependencies, builds the Saleor API image, scans the image for vulnerabilities, tests the application, and pushes the completed image to Docker Hub. The Docker images are published with develop, staging, and production tags.

For the cloud deployment, I used Amazon ECS with AWS Fargate. Fargate runs the containers without requiring me to create or manage a traditional EC2 server. I created an ECS cluster, task definition, service, networking configuration, security groups, and an Application Load Balancer.

The Application Load Balancer receives incoming web traffic and forwards it to the Saleor API container. CloudWatch captures logs from the deployed containers, which helped me troubleshoot application and database errors.

One major issue was that the Saleor API initially returned HTTP 500 errors because the required Django database tables did not exist. I corrected this by changing the container startup process so it runs database migrations before starting Gunicorn. After the migrations completed, the application could start correctly and the ECS task became successful.

This project gave me hands-on experience with Docker, Docker Compose, GitHub Actions, Docker Hub, vulnerability scanning, Amazon ECS, Fargate, load balancing, networking, logging, health checks, database migrations, and production troubleshooting.
