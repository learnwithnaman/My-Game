# 🎮 2048 Game App - Dockerized Application

## 📌 Project Overview

This project demonstrates a **complete Full Stack Game application** built with modern web technologies and containerized using Docker. The application features a **React.js frontend** and **Node.js + Express backend**, orchestrated seamlessly with Docker Compose for a portable, scalable, and DevOps-ready deployment.

The entire application can be spun up with a single command, making it ideal for **development, testing, and production-ready deployments**.

![UI](diagram/ui.png)

---

## 🏗️ Project Structure

```
2048-Game/
│
├── client/                     # React Frontend Application
│   ├── public/                 # Static assets
│   ├── src/                    # React source code
│   │   ├── components/         # Reusable UI components
│   │   ├── pages/              # Application pages
│   │   ├── services/           # API service layer
│   │   └── App.js              # Main application component
│   ├── .env                    # Environment variables
│   ├── Dockerfile              # Frontend container configuration
│   └── package.json            # Frontend dependencies
│
├── server/                     # Node.js Backend Application
│   ├── controllers/            # Request handlers
│   ├── routes/                 # API route definitions
│   ├── middleware/             # Custom middleware
│   ├── .env                    # Environment variables
│   ├── Dockerfile              # Backend container configuration
│   ├── package.json            # Backend dependencies
│   └── server.js               # Application entry point
│
├── docker-compose.yml          # Multi-container orchestration
└── README.md                   # Project documentation
```

![Structure](diagram/structure.png)

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         User Browser                        │
│                    (http://localhost:3000)                  │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Docker Container Network                  │
│                                                             │
│  ┌─────────────────────┐        ┌─────────────────────┐    │
│  │   React Frontend    │───────▶│   Node.js Backend   │    │
│  │   Container         │  API   │   Container         │    │
│  │                     │ Calls  │                     │    │
│  │   Port: 3000        │◀───────│   Port: 5000        │    │
│  │   Exposed to Host   │  JSON  │   Internal Only     │    │
│  └─────────────────────┘        └─────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      Application Flow                        │
│                                                             │
│  1. User interacts with React UI                            │
│  2. Frontend makes API call to http://server:5000          │
│  3. Backend processes request & returns JSON response       │
│  4. Frontend updates UI with received data                  │
└─────────────────────────────────────────────────────────────┘
```

![Architecture](diagram/architecture.png)

---

## 🛠️ Technologies Used

| Technology | Purpose |
|------------|---------|
| **React.js** | Frontend UI framework |
| **Node.js** | Backend JavaScript runtime |
| **Express.js** | Backend web framework |
| **Docker** | Containerization platform |
| **Docker Compose** | Multi-container orchestration |
| **Axios** | HTTP client for API calls |


## 🐳 Docker Configuration

#### 📦 Client Dockerfile

```dockerfile
# Multi-stage build for optimized image size
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files for dependency caching
COPY package*.json ./
RUN npm ci --only=production

# Copy application source
COPY . .

# Build the application (if using production build)
# RUN npm run build

# Production stage
FROM node:20-alpine

WORKDIR /app

# Copy node_modules and application files
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app ./

# Expose React development port
EXPOSE 3000

# Start the development server
CMD ["npm", "start"]
```

#### 📦 Server Dockerfile

```dockerfile
# Multi-stage build for optimized image size
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files for dependency caching
COPY package*.json ./
RUN npm ci --only=production

# Copy application source
COPY . .

# Production stage
FROM node:20-alpine

WORKDIR /app

# Install curl for health checks
RUN apk add --no-cache curl

# Copy node_modules and application files
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app ./

# Expose backend port
EXPOSE 5000

# Health check to ensure server is running
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1

# Start the server
CMD ["npm", "start"]
```

## 🧪 Docker Compose Configuration

```yaml
version: "3.8"

services:
  # Backend Node.js Server
  server:
    build:
      context: ./server
      dockerfile: Dockerfile
    container_name: game_server
    ports:
      - "5000:5000"
    restart: unless-stopped
    environment:
      - NODE_ENV=development
      - PORT=5000
    env_file:
      - ./server/.env
    networks:
      - game-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s

  # Frontend React Application
  client:
    build:
      context: ./client
      dockerfile: Dockerfile
    container_name: game_client
    ports:
      - "3000:3000"
    restart: unless-stopped
    depends_on:
      - server
    environment:
      - REACT_APP_API_URL=http://server:5000
    env_file:
      - ./client/.env
    networks:
      - game-network
    volumes:
      - ./client:/app
      - /app/node_modules

# Define custom network for inter-container communication
networks:
  game-network:
    driver: bridge
```

## 🚀 How to Run the Project?

#### Step 1: Prerequisites

Ensure you have the following installed on your system:
- **Git** - For cloning the repository
- **Docker** - For containerization
- **Docker Compose** - For orchestration

#### Step 2: Clone Repository

```bash
git clone <your-repo-url>
cd 2048-Game
```

#### Step 3: Install Docker & Docker Compose (Ubuntu/Debian)

```bash
# Update package index
sudo apt update -y
sudo apt upgrade -y

# Install Docker
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# Add current user to docker group (to avoid sudo)
sudo usermod -aG docker $USER
newgrp docker

# Verify Docker installation
docker --version

# Install Docker Compose
sudo apt install docker-compose-v2 -y

# Verify Docker Compose
docker compose version
```

#### Step 4: Configure Environment Variables

**Server Environment (.env)**
```env
PORT=5000
NODE_ENV=development
```

**Client Environment (.env)**
```env
# For local development outside Docker
REACT_APP_API_URL=http://localhost:5000

# For Docker container (uncomment when using Docker Compose)
# REACT_APP_API_URL=http://server:5000
```

#### Step 5: Build and Run Application

```bash
# Build and start all containers
docker compose up --build

# Run in detached mode (background)
docker compose up --build -d

# View logs
docker compose logs -f
```

#### Step 6: Verify Running Containers

```bash
# Check container status
docker compose ps

# Check running containers
docker ps

# View container logs
docker logs game_server
docker logs game_client
```

#### Step 7: Access the Application

| Component | URL | Description |
|-----------|-----|-------------|
| **Frontend** | http://localhost:3000 | React game interface |
| **Backend API** | http://localhost:5000 | Node.js API endpoint |
| **Health Check** | http://localhost:5000/health | Backend health status |


## 🎯 Conclusion

This project represents a **production-ready, containerized full-stack application** that can be deployed anywhere Docker runs. The clean separation of concerns, optimized Docker images, and comprehensive orchestration.

*Built with ❤️ by **Naman Pandey** | DevOps & Cloud Engineer* 🚀
