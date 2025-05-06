# Githubactions
CI\CD

# Sample Web Application - CI/CD with GitHub Actions 🚀

This project showcases a full CI/CD pipeline using **GitHub Actions**, **DockerHub**, and **SSH-based deployment**. It includes linting, testing, Docker image build and push, deployment with manual approval, and rollback logic.

---

## 🗂 Project Structure

sample-webapp/
├── .github/
│ └── workflows/
│ └── ci-cd.yml
├── app/
│ └── main.py
├── Dockerfile
├── requirements.txt
├── README.md


---

## ⚙️ GitHub Actions CI/CD Workflow

The CI/CD pipeline is triggered **on every push to `main`** and performs the following:

1. ✅ **Linting and Unit Testing** with `flake8` and `pytest`
2. 🐳 **Build and Push Docker Image** to DockerHub
3. 🚀 **Manual Approval & Deployment** via GitHub Environment
4. 🔁 **Rollback Logic** in case of failure

---

## 🔐 Required Secrets

Set these in your GitHub repository:

| Secret Name          | Description                       |
|----------------------|-----------------------------------|
| `DOCKERHUB_USERNAME` | Your DockerHub username           |
| `DOCKERHUB_TOKEN`    | Your DockerHub token or password  |

Go to:  
**Repo → Settings → Secrets and Variables → Actions → New repository secret**

---

## 🛠 Sample Workflow File (`.github/workflows/ci-cd.yml`)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]

env:
  IMAGE_NAME: yourdockerhubusername/sample-webapp
  DEPLOY_ENV: production

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install flake8 pytest

      - name: Run linter
        run: flake8 .

      - name: Run unit tests
        run: pytest

  build-and-push:
    needs: lint-and-test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker image
        run: docker build -t $IMAGE_NAME:latest .

      - name: Push Docker image
        run: docker push $IMAGE_NAME:latest

  deploy:
    name: Deploy to ${{ env.DEPLOY_ENV }}
    needs: build-and-push
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://your-app-url.com

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Deploy app (example using SSH)
        run: |
          ssh user@your-server "
            docker pull $IMAGE_NAME:latest &&
            docker stop webapp || true &&
            docker rm webapp || true &&
            docker run -d --name webapp -p 80:80 $IMAGE_NAME:latest
          "
        env:
          IMAGE_NAME: ${{ env.IMAGE_NAME }}
        continue-on-error: true

      - name: Rollback on failure
        if: failure()
        run: |
          echo "Deployment failed. Rolling back..."
          ssh user@your-server "
            docker stop webapp || true &&
            docker rm webapp || true &&
            docker run -d --name webapp -p 80:80 $IMAGE_NAME:previous
          "

=============================================================================

Bonus Features
✅ Manual Approval via GitHub Environments
This workflow uses GitHub Environments for production deployment, which allows for manual approval before deploying.

The deploy job is tied to the production environment:

yaml
Copy
Edit
environment:
  name: production
  url: https://your-app-url.com


✅ Rollback Logic on Deployment Failure
The workflow includes automatic rollback logic in case the deployment fails:

yaml
Copy
Edit
- name: Rollback on failure
  if: failure()
  run: |
    echo "Deployment failed. Rolling back..."
    ssh user@your-server "
      docker stop webapp || true &&
      docker rm webapp || true &&
      docker run -d --name webapp -p 80:80 $IMAGE_NAME:previous
    "
If the deployment step fails (e.g., due to container crash or port conflict), the pipeline:

Stops and removes the failed container.

Launches a previous version tagged as :previous.

🛠 Important: You must manually push or tag a working Docker image as :previous after a successful deployment:

bash
Copy
Edit
docker tag yourdockerhubusername/sample-webapp:latest yourdockerhubusername/sample-webapp:previous
docker push yourdockerhubusername/sample-webapp:previous
