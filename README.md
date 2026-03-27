# Coworking Analytics Microservice Deployment

This project deploys a Python analytics application and a PostgreSQL database to Kubernetes on AWS. The application is containerized with Docker, built automatically with AWS CodeBuild, stored in Amazon ECR, and deployed to a Kubernetes cluster using Kubernetes manifests.

## Project Overview

The deployment consists of two main services:

- A PostgreSQL database running inside the Kubernetes cluster
- A Python Flask analytics application exposed through a Kubernetes LoadBalancer service

The application reads configuration from a Kubernetes ConfigMap and sensitive values from a Kubernetes Secret. Kubernetes health and readiness probes are used to verify that the application is running correctly before it receives traffic.

## Build and Release Process

The build pipeline uses AWS CodeBuild to build the Docker image from the application source code. After a successful build, CodeBuild authenticates with Amazon ECR and pushes the image to the repository.

Amazon ECR acts as the image registry for versioned application builds. The Kubernetes deployment references a specific image tag so releases are explicit and repeatable.

To release a new version of the application:

1. Update the application source code
2. Commit and push the changes to GitHub
3. Build and push a new Docker image to Amazon ECR with a new semantic version tag
4. Update the image tag in `deployment/coworking.yaml`
5. Apply the Kubernetes manifests with `kubectl apply -f`
6. Verify the rollout with `kubectl get pods`, `kubectl get svc`, and application endpoint checks

## Docker

The application is packaged using a Dockerfile based on a Python runtime image. Dependencies are installed during the image build, and the container starts the Flask application on port `5153`.

The image is versioned using semantic versioning. The deployed version for this project is:

`767701879863.dkr.ecr.us-east-1.amazonaws.com/coworking-analytics:1.0.0`

## AWS CodeBuild

AWS CodeBuild is used to automate the container build and push process. The build configuration authenticates to Amazon ECR, builds the Docker image, tags it, and pushes it to the repository.

This approach separates build concerns from deployment concerns and makes it easier to produce consistent container images for Kubernetes.

## Kubernetes Deployment

The Kubernetes resources are defined as YAML manifests and include:

- Deployment for the analytics application
- Service for the analytics application
- ConfigMap for non-sensitive environment variables
- Secret for sensitive database credentials
- Deployment and Service for PostgreSQL

The analytics application is exposed externally using a `LoadBalancer` service on port `5153`. PostgreSQL is exposed internally within the cluster using a `ClusterIP` service.

Health monitoring is implemented with:

- Liveness probe: `/health_check`
- Readiness probe: `/readiness_check`

These probes help Kubernetes determine when the container is healthy and ready to serve traffic.

## Configuration Management

Application configuration is separated into two types:

- Non-sensitive configuration in a ConfigMap:
  - `DB_HOST`
  - `DB_PORT`
  - `DB_NAME`
  - `DB_USERNAME`

- Sensitive configuration in a Secret:
  - `DB_PASSWORD`

This keeps deployment configuration flexible and avoids hardcoding credentials directly in the application container image.

## Validation

The deployment was validated by confirming:

- Kubernetes pods are in the `Running` state
- Services are created successfully
- The external application endpoint responds correctly
- The health check endpoint returns `ok`
- The daily usage report endpoint returns JSON data from PostgreSQL

Example endpoints:

- `/health_check`
- `/api/reports/daily_usage`

## Files Submitted

The submission includes:

- `Dockerfile`
- Kubernetes YAML configuration files used for deployment
- `README.md`

## Standout Suggestions

### Resource allocation
Reasonable CPU and memory requests/limits should be added to the Kubernetes deployment so the scheduler can place pods more efficiently and to reduce the risk of noisy-neighbor issues. For this application, modest values would likely be enough because the service is lightweight and performs simple analytics queries.

### Best EC2 instance type
A small general-purpose instance type such as `t3.small` or `t3.medium` would be a practical choice for this application because the workload is lightweight and does not require high compute power. These instance types provide a good balance of cost, memory, and CPU for a small microservice and database deployment.

### Cost savings
Costs can be reduced by using smaller instance types, scaling down non-production environments when not in use, and avoiding over-provisioning CPU and memory. Additional savings can come from cleaning up unused load balancers, container images, and other AWS resources after testing.
