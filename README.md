# Soul Start Up

A static website project containerized using Docker and Nginx, with a local AWS-compatible environment powered by Floci.

## 1. Project Overview

This project demonstrates how to:

- Serve a static website using Nginx.
- Containerize the application using Docker.
- Run the application using Docker Compose.
- Create a local AWS ECR repository using Floci.
- Build, tag and push Docker images to ECR.
- Prepare the project for automated CI/CD using GitHub Actions.

## 2. Technology Stack

| Technology | Purpose |
|---|---|
| HTML, CSS, JavaScript | Website frontend |
| Nginx | Web server |
| Docker | Application containerization |
| Docker Compose | Container management |
| Floci | Local AWS service emulation |
| AWS CLI | Manage local AWS services |
| Amazon ECR (emulated) | Docker image registry |
| GitHub Actions | CI/CD automation (planned) |

## 3. Project Structure

```text
soul-start-up/
│
├── fast foward/
│   ├── fonts/
│   ├── config.js
│   ├── favicon.svg
│   ├── font.css
│   ├── icons.js
│   └── index.html
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── Dockerfile
├── compose.yaml
├── .dockerignore
├── .gitignore
└── README.md
```

## 4. Prerequisites

Install the following tools on your machine:

- Git
- Docker Desktop
- AWS CLI v2
- Floci CLI

Verify installations:

```bash
git --version
docker --version
docker compose version
aws --version
floci --version
```

## 5. Clone the Repository

```bash
git clone https://github.com/Soul-hacker-in/soul-start-up.git
```

Navigate to the project directory:

```bash
cd soul-start-up
```

## 6. Docker Configuration

### Dockerfile

The Dockerfile defines how the website Docker image is created.

```dockerfile
FROM nginx:alpine

COPY ["fast foward/", "/usr/share/nginx/html/"]

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Explanation:

- `FROM nginx:alpine`: Uses lightweight Nginx as the base image.
- `COPY`: Copies website files into the Nginx web directory.
- `EXPOSE 80`: Documents the web server port.
- `CMD`: Starts the Nginx server.

### Docker Compose

The `compose.yaml` file defines the application container.

```yaml
services:
  website:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: soul-start-up
    ports:
      - "8080:80"
    restart: unless-stopped
```

Port mapping:

- Host port: 8080
- Container port: 80

## 7. Build and Run the Application

Build the Docker image:

```bash
docker compose build
```

Start the application:

```bash
docker compose up -d
```

Check running containers:

```bash
docker ps
```

Access the website:

http://localhost:8080

### Stop the Application

```bash
docker compose down
```

### Rebuild After Code Changes

```bash
docker compose build --no-cache
docker compose up -d
```

## 8. Configure Floci (Local AWS Environment)

Floci provides a local AWS-compatible environment for development and testing without creating real AWS resources.

Start Floci:

```bash
floci start
```

Check the environment:

```bash
floci doctor
```

Configure the current terminal:

```bash
eval "$(floci env)"
```

Verify the endpoint and region:

```bash
echo $AWS_ENDPOINT_URL
echo $AWS_DEFAULT_REGION
```

Expected configuration:

```text
AWS_ENDPOINT_URL=http://localhost.floci.io:4566
AWS_DEFAULT_REGION=us-east-1
```

Floci endpoint:

http://localhost:4566

## 9. Verify AWS CLI Connectivity

Check AWS CLI:

```bash
aws --version
```

Verify AWS identity:

```bash
aws sts get-caller-identity
```

List S3 buckets:

```bash
aws s3 ls
```

The local Floci account uses:

```text
Account ID: 000000000000
Region: us-east-1
```

Note: These are local emulated AWS resources, not real AWS account resources.

## 10. Create ECR Repository

Amazon Elastic Container Registry (ECR) stores Docker images.

Create the repository:

```bash
aws ecr create-repository \
  --repository-name soul-start-up \
  --region us-east-1
```

Check the repository:

```bash
aws ecr describe-repositories \
  --repository-names soul-start-up \
  --region us-east-1
```

Repository URI:

```text
000000000000.dkr.ecr.us-east-1.localhost:4566/soul-start-up
```

## 11. Authenticate Docker with ECR

Get the ECR login password and authenticate Docker:

```bash
aws ecr get-login-password \
  --region us-east-1 | docker login \
  --username AWS \
  --password-stdin 000000000000.dkr.ecr.us-east-1.localhost:4566
```

Expected output:

```text
Login Succeeded
```

## 12. Build Docker Image

Build the application image:

```bash
docker build -t soul-start-up:latest .
```

Verify the image:

```bash
docker images
```

## 13. Tag Docker Image

Tag the image with the Floci ECR repository URI:

```bash
docker tag soul-start-up:latest \
000000000000.dkr.ecr.us-east-1.localhost:4566/soul-start-up:latest
```

Docker image naming convention:

```text
REGISTRY/REPOSITORY:TAG
```

For this project:

- Registry: `000000000000.dkr.ecr.us-east-1.localhost:4566`
- Repository: `soul-start-up`
- Tag: `latest`

## 14. Push Docker Image to ECR

Upload the image:

```bash
docker push \
000000000000.dkr.ecr.us-east-1.localhost:4566/soul-start-up:latest
```

The image is now stored in the local Floci ECR repository.

## 15. Verify Uploaded Docker Image

List images:

```bash
aws ecr list-images \
  --repository-name soul-start-up \
  --region us-east-1
```

Expected result:

```json
{
  "imageIds": [
    {
      "imageTag": "latest"
    }
  ]
}
```

## 16. Complete Docker and ECR Workflow

The current manual deployment preparation process:

```text
Website Source Code
        |
        v
   Dockerfile
        |
        v
   Docker Build
        |
        v
 soul-start-up:latest
        |
        v
    Docker Tag
        |
        v
     Docker Login
        |
        v
     Docker Push
        |
        v
     Floci ECR
        |
        v
  Image Stored Successfully
```

## 17. Planned CI/CD Automation

The next development phase is to automate the manual Docker and ECR operations using GitHub Actions.

Target workflow:

```text
Developer
    |
    v
Git Push
    |
    v
GitHub Repository
    |
    v
GitHub Actions
    |
    v
Self-hosted Runner (Mac)
    |
    v
Docker Image Build
    |
    v
Push Image to Floci ECR
    |
    v
Pull Updated Image
    |
    v
Restart Docker Container
    |
    v
Updated Website
```

### Planned Automation

1. Trigger the pipeline whenever code is pushed to the main branch.
2. Build the Docker image automatically.
3. Tag the image with a version or commit SHA.
4. Push the image to Floci ECR.
5. Deploy the updated container using Docker Compose.

**Note:** Automated GitHub Actions deployment is the next implementation phase. The current Docker build, local website execution and Floci ECR image push have been completed manually.

## 18. Useful Commands Reference

| Command | Description |
|---|---|
| `floci start` | Start local AWS emulator |
| `floci doctor` | Check Floci environment |
| `eval "$(floci env)"` | Configure Floci environment |
| `docker ps` | List running containers |
| `docker images` | List local Docker images |
| `docker compose build` | Build application image |
| `docker compose up -d` | Start application |
| `docker compose down` | Stop application |
| `aws ecr describe-repositories` | List ECR repositories |
| `aws ecr list-images` | List repository images |
| `docker push IMAGE` | Upload image to ECR |

---

## 19. Important Notes

- Floci is used for local AWS service emulation.
- Docker Desktop and Floci must be running for local operations.
- Run `eval "$(floci env)"` in each new terminal session before using the local AWS CLI.
- The local ECR repository is not accessible as a real AWS ECR repository.
- The GitHub self-hosted runner must run on your Mac to access local Floci.
- Do not commit AWS credentials, secrets or local environment files to GitHub.

## Repository

GitHub: https://github.com/Soul-hacker-in/soul-start-up
