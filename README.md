# Hello Docker

A very simple Docker project that serves a static **Hello World** HTML page using Nginx.

This project is designed as a beginner-friendly example of how Docker images can be built locally and automatically built/pushed using GitHub Actions.

---

## Project Structure

```text
hello-docker/
├── index.html
├── Dockerfile
└── .github/
    └── workflows/
        └── docker-build.yaml
```

---

## What This App Does

The app serves a basic HTML page:

```html
<h1>Hello World from Docker!</h1>
```

Docker packages this HTML file into an Nginx web server image.

---

## Files

### `index.html`

```html
<!DOCTYPE html>
<html>
<head>
  <title>Hello Docker</title>
</head>
<body>
  <h1>Hello World from Docker!</h1>
</body>
</html>
```

### `Dockerfile`

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

---

## Run Locally

Build the Docker image:

```bash
docker build -t hello-docker .
```

Run the container:

```bash
docker run -p 8080:80 hello-docker
```

Open your browser and visit:

```text
http://localhost:8080
```

You should see:

```text
Hello World from Docker!
```

---

## GitHub Actions Build

This project can automatically build and push the Docker image when code is pushed to the `main` branch.

Create this workflow file:

```text
.github/workflows/docker-build.yaml
```

```yaml
name: Build Docker Image

on:
  push:
    branches:
      - main

env:
  IMAGE_NAME: ghcr.io/${{ github.repository_owner }}/hello-docker

jobs:
  build:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            ${{ env.IMAGE_NAME }}:latest
            ${{ env.IMAGE_NAME }}:${{ github.sha }}
```

After pushing to `main`, GitHub Actions will:

1. Check out the repository
2. Build the Docker image
3. Push the image to GitHub Container Registry

---

## Run Image from GitHub Container Registry

After the image is pushed, you can run it on a server:

```bash
docker login ghcr.io
```

Then run the container:

```bash
docker run -d \
  --name hello-docker \
  --restart unless-stopped \
  -p 80:80 \
  ghcr.io/YOUR_GITHUB_USERNAME/hello-docker:latest
```

Replace:

```text
YOUR_GITHUB_USERNAME
```

with your actual GitHub username or organization name.

---

## Stop the Container

```bash
docker stop hello-docker
```

Remove the container:

```bash
docker rm hello-docker
```

---

## Useful Docker Commands

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

List Docker images:

```bash
docker images
```

View container logs:

```bash
docker logs hello-docker
```

Rebuild the image:

```bash
docker build -t hello-docker .
```

---

## Production Flow

The basic production-style flow is:

```text
Push code to GitHub
        ↓
GitHub Actions builds Docker image
        ↓
Image is pushed to GitHub Container Registry
        ↓
Server pulls and runs the image
```

---



