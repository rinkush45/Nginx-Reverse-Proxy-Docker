# Nginx-Reverse-Proxy-Docker Setup

---

## 📺 Video Setup Guide

You can watch the full setup process in the video below:

<video src="media/setup-demo.mp4" controls width="700">
  Your browser does not support the video tag.
</video>

---

## 1. Project Structure

```
Nginx-Reverse-Proxy-Docker
│
|
├── media/
│   └── setup-demo.mp4
|
├── nginx/
│   ├── nginx.conf
│   └── Dockerfile
│
├── service_1/
│   ├── main.go
│   ├── Dockerfile
│   ├── go.mod
│   └── README.md
│
├── service_2/
│   ├── app.py
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── pyproject.toml
│   ├── uv.lock
│   ├── .python-version
│   └── README.md
│
└── docker-compose.yaml
```

---

## 2. Service-by-Service Breakdown

### A. nginx (Reverse Proxy)
- **nginx.conf**: Configures Nginx as a reverse proxy, routing incoming HTTP requests to the appropriate backend service.
- **Dockerfile**: Builds a Docker image for Nginx using the provided configuration.
- **Role**: Acts as the entry point for all external traffic, forwarding requests to the correct backend service.

### B. service_1 (Go Service)
- **main.go**: The main Go application, likely starting a web server on port 8001.
- **Dockerfile**: Builds a Docker image for the Go service.
- **go.mod**: Go module definition, listing dependencies.
- **README.md**: Documentation for the service.
- **Role**: Provides a backend service, possibly a REST API, running on port 8001.

### C. service_2 (Python Service)
- **app.py**: The main Python application, likely a web server running on port 8002.
- **requirements.txt**: Lists Python dependencies.
- **pyproject.toml** and **uv.lock**: Used for Python dependency management.
- **Dockerfile**: Builds a Docker image for the Python service.
- **README.md**: Documentation for the service.
- **Role**: Provides another backend service, running on port 8002.

---

## 3. Orchestration: docker-compose.yaml

- **nginx**:
  - Built from ./nginx.
  - Exposes port 8080 on the host, mapped to port 80 in the container.
  - Depends on service_1 and service_2.
  - Connected to the app-network.

- **service_1**:
  - Built from ./service_1.
  - Exposes port 8001 (internal only).
  - Healthcheck: Uses curl to check http://localhost:8001.
  - Connected to the app-network.

- **service_2**:
  - Built from ./service_2.
  - Exposes port 8002 (internal only).
  - Healthcheck: Uses curl to check http://localhost:8002.
  - Connected to the app-network.

- **Network**: All services are on a custom bridge network called app-network, allowing them to communicate by service name.

---

## 4. How the System Works (Step-by-Step)

1. **Build and Start**: Running `docker-compose up` builds images for nginx, service_1, and service_2, and starts containers for each.
2. **Service Startup**:
   - service_1 starts a Go web server on port 8001.
   - service_2 starts a Python web server on port 8002.
   - nginx starts and loads its configuration from nginx.conf.
3. **Healthchecks**: docker-compose monitors service_1 and service_2 using curl to ensure they are running.
4. **Request Flow**:
   - External requests come to localhost:8080 (host) → forwarded to nginx:80 (container).
   - nginx, based on nginx.conf, routes requests to either service_1:8001 or service_2:8002, depending on the request path or host.
   - service_1 and service_2 process the requests and return responses via nginx.
5. **Networking**: All services communicate over the app-network, using their service names as hostnames.

---

## 5. Summary Table

| Service     | Language | Port | Dockerized | Role                | Exposed to Host | Internal Name |
|-------------|----------|------|------------|---------------------|-----------------|--------------|
| nginx       | Nginx    | 80   | Yes        | Reverse Proxy       | 8080            | nginx        |
| service_1   | Go       | 8001 | Yes        | Backend API         | No              | service_1    |
| service_2   | Python   | 8002 | Yes        | Backend API         | No              | service_2    |

---

## 6. How to Run the Project

1. Ensure Docker and Docker Compose are installed.
2. In the project root, run:
   ```
   docker-compose up --build
   ```
3. Access the system at: [http://localhost:8080](http://localhost:8080)
4. Nginx will route requests to the appropriate backend service.

---