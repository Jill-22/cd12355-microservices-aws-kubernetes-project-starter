# Coworking Analytics Microservice Deployment

This project deploys a Python analytics application and a PostgreSQL database to Kubernetes on AWS.  
The application is containerized with Docker, built with AWS CodeBuild, stored in Amazon ECR, and deployed with Kubernetes manifests.

The deployment includes a Flask analytics service and a PostgreSQL database service.  
The analytics service is exposed with a Kubernetes `LoadBalancer`, while PostgreSQL is exposed internally with a `ClusterIP` service.

The application uses a ConfigMap for non-sensitive environment variables and a Secret for sensitive values.  
Health monitoring is configured with liveness and readiness probes at `/health_check` and `/readiness_check`.

AWS CodeBuild builds the Docker image and pushes it to Amazon ECR.  
The deployed image version for this project is `767701879863.dkr.ecr.us-east-1.amazonaws.com/coworking-analytics:1.0.0`.

To release a new version, update the source code, build and push a new image to ECR, update the image tag in `deployment/coworking.yaml`, and apply the manifests with `kubectl apply -f`.  
The deployment was validated by confirming that pods were running, services were created, and the application endpoints responded successfully.

The health check endpoint returned `ok`.  
The daily usage report endpoint returned JSON data from PostgreSQL.

The repository includes the `Dockerfile`, Kubernetes YAML files, and this `README.md`.  
Project screenshots are stored in the `screenshots/` folder.

Recommended screenshots include CodeBuild success, ECR image tags, `kubectl get svc`, `kubectl get pods`, `kubectl describe svc postgresql-service`, `kubectl describe deployment coworking`, and CloudWatch Container Insights logs.  

## Standout Suggestions

A small general-purpose EC2 instance such as `t3.small` or `t3.medium` is a practical choice for this workload.  
Cost savings can come from using smaller instances, scaling down non-production environments, and removing unused AWS resources after testing.
