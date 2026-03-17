# Docker Setup Guide

This guide explains how to build, run, and manage the Docker containers for this project using **Docker Compose**.

## Prerequisites

Make sure you have the following installed:

* Docker
* Docker Compose

You can verify installation with:

```bash
docker --version
docker-compose --version
```

---

# Project Setup

## 1. Build the Docker Image

Build the Docker image defined in the `docker-compose.yml` file.

```bash
docker-compose build
```

---

## 2. Run the Container

Start the container using Docker Compose.

```bash
docker-compose up
```

---

## 3. Build and Run in Background (Detached Mode)

Build the image and start containers in the background.

```bash
docker-compose up -d
```

---

## 4. Rebuild and Run

Rebuild the image and start the containers.

```bash
docker-compose up --build
```

---

# Managing Containers

## Stop Containers

Stop and remove containers defined in the compose file.

```bash
docker-compose down
```

---

## Stop and Remove Everything (Including Volumes)

This will stop containers and remove volumes.

```bash
docker-compose down -v
```

---

# Monitoring Containers

## View Logs

View real-time logs from the containers.

```bash
docker-compose logs -f
```

---

## View Running Containers

List currently running Docker containers.

```bash
docker ps
```

---

# Access the Container Shell

Open a shell session inside the running container.

```bash
docker exec -it 2048-game sh
```

---

# Quick Start (Recommended)

Run the following command to **build and start the project in one step**:

```bash
docker-compose up --build
```