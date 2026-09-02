# Todo App Docker - Automated CI/CD Pipeline

A containerized three-tier Todo application with an automated Jenkins CI/CD pipeline. The project demonstrates how a React frontend, Flask REST API, and MongoDB database can be built, published, and deployed as a reproducible Docker Compose stack.

## Features

- Create, list, update, and delete Todo tasks.
- React-based user interface.
- Flask REST API with CORS support.
- MongoDB persistence using a named Docker volume.
- Separate frontend, backend, and database containers.
- Docker Compose orchestration.
- Jenkins pipeline-as-code through `Jenkinsfile`.
- Docker Hub image publication.
- Automated replacement of the previously deployed stack.

## Architecture

```text
User browser
    |
    v
React frontend :3000
    |
    v
Flask REST API :5000
    |
    v
MongoDB :27017
    |
    v
mongo-data volume
```

The application is divided into three services:

| Service | Technology | Port | Purpose |
| --- | --- | ---: | --- |
| `frontend` | React / Node.js | `3000` | Todo user interface |
| `backend` | Python / Flask | `5000` | REST API and application logic |
| `mongo` | MongoDB | `27017` | Persistent Todo storage |

## Technology Stack

- React 16
- Node.js 18 Alpine container
- Python 3.9 Slim container
- Flask and Flask-PyMongo
- MongoDB
- Docker and Docker Compose
- Jenkins
- GitHub
- Docker Hub

## Repository Structure

```text
.
├── client/                 # React frontend
│   ├── public/
│   ├── src/
│   ├── Dockerfile
│   └── package.json
├── server/                 # Flask backend
│   ├── Dockerfile
│   ├── mongo.py
│   └── requirements.txt
├── docker-compose.yml      # Three-service application stack
└── Jenkinsfile             # Build, publish, and deployment pipeline
```

## Prerequisites

For local Docker deployment:

- Git
- Docker Engine or Docker Desktop
- Docker Compose

For CI/CD deployment:

- A Jenkins server or agent with Git, Docker, and Docker Compose available.
- Access to the Docker daemon from Jenkins.
- A Docker Hub account.
- Jenkins credentials for Docker Hub.

Node.js, Python, and MongoDB do not need to be installed directly when the application is run with Docker.

## Quick Start with Docker Compose

### 1. Clone the repository

```bash
git clone https://github.com/amine-rahbani/todo-app-docker.git
cd todo-app-docker
```

### 2. Build and start the application

With Docker Compose v2:

```bash
docker compose up -d --build
```

If your environment uses the older standalone Compose command:

```bash
docker-compose up -d --build
```

### 3. Open the services

- Frontend: <http://localhost:3000>
- Backend API: <http://localhost:5000/api/tasks>
- MongoDB: `localhost:27017`

### 4. Check the containers

```bash
docker compose ps
```

### 5. View logs

```bash
docker compose logs -f
```

Use `Ctrl+C` to stop following the logs without stopping the containers.

### 6. Stop the application

```bash
docker compose down
```

The `mongo-data` named volume preserves Todo records when containers are stopped or recreated.

To remove the stack and its stored MongoDB data:

```bash
docker compose down -v
```

> [!WARNING]
> The `-v` option permanently removes the MongoDB volume and its stored Todo data.

## REST API

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/tasks` | Return all tasks |
| `POST` | `/api/task` | Create a task |
| `PUT` | `/api/task/{id}` | Update a task |
| `DELETE` | `/api/task/{id}` | Delete a task |

Example request for creating a task:

```bash
curl -X POST http://localhost:5000/api/task \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn Docker Compose"}'
```

Example response:

```json
{
  "result": {
    "title": "Learn Docker Compose"
  }
}
```

## Docker Configuration

The Compose file builds and starts three services:

- `backend` builds from `server/` and publishes the Flask API on port `5000`.
- `frontend` builds from `client/` and publishes the React development server on port `3000`.
- `mongo` uses the official MongoDB image and stores data in the `mongo-data` volume.

The backend connects to MongoDB through the internal Compose hostname `mongo`. Docker Compose creates the shared application network automatically.

The configured Docker Hub images are:

```text
aminerahbani/todo-backend:latest
aminerahbani/todo-frontend:latest
```

Change the image names in `docker-compose.yml` and `DOCKER_USER` in `Jenkinsfile` if you deploy through another Docker Hub account.

## Jenkins CI/CD Pipeline

The `Jenkinsfile` defines the following workflow:

```text
Checkout source
      |
      v
Build Docker images
      |
      v
Authenticate with Docker Hub
      |
      v
Push frontend and backend images
      |
      v
Stop the previous Compose stack
      |
      v
Deploy the new stack
      |
      v
Log out from Docker Hub
```

### Jenkins credential

Create a Jenkins username/password credential containing your Docker Hub username and access token or password.

Use this credential ID:

```text
docker-hub-credentials
```

The ID must match the following `Jenkinsfile` setting:

```groovy
DOCKER_CREDS_ID = 'docker-hub-credentials'
```

### Create the Jenkins pipeline

1. In Jenkins, select **New Item**.
2. Enter a job name such as `deploy-todo-app`.
3. Select **Pipeline**.
4. Under **Pipeline**, choose **Pipeline script from SCM**.
5. Select **Git** as the SCM.
6. Enter the GitHub repository URL.
7. Select the repository credentials if required.
8. Set the branch to `*/main`.
9. Set the script path to `Jenkinsfile`.
10. Save the job and select **Build Now**.

### Automatic GitHub trigger

The pipeline can be triggered automatically using a GitHub webhook or Jenkins source-control polling. The Jenkins server must be reachable by GitHub for webhook delivery.

Do not expose a development Jenkins server to the public internet without authentication, TLS, access controls, and appropriate hardening.

### Docker access from Jenkins

Jenkins must be able to execute `docker` and `docker-compose`. When Jenkins runs inside a container, one approach is to provide the Docker CLI and mount the host Docker socket:

```text
/var/run/docker.sock:/var/run/docker.sock
```

> [!CAUTION]
> Access to the Docker socket provides extensive control over the Docker host. Use a dedicated, trusted Jenkins environment and restrict access to the Jenkins server.

## Current Pipeline Scope

The current pipeline performs:

- Source checkout
- Image builds
- Docker Hub authentication
- Image publication
- Compose deployment
- Docker Hub logout

Automated unit, integration, security, and quality checks are not currently defined in the supplied `Jenkinsfile`. These should be added before treating the pipeline as production-ready.

## Troubleshooting

### Port already in use

Stop an existing copy before deploying another:

```bash
docker compose down
```

Then check active containers:

```bash
docker ps
```

The application needs ports `3000`, `5000`, and `27017` to be available.

### Backend cannot reach MongoDB

Check the backend and database logs:

```bash
docker compose logs backend mongo
```

The MongoDB hostname inside the Compose network must remain `mongo` unless the backend configuration is updated.

### Jenkins cannot run Docker

Confirm that:

- Docker is installed and running on the Jenkins host.
- Jenkins has permission to access the Docker daemon.
- The Docker CLI is available to the Jenkins agent.
- `docker-compose` is available because the current `Jenkinsfile` uses that command.

### Docker Hub authentication fails

Confirm that:

- The Jenkins credential ID is `docker-hub-credentials`.
- The stored username belongs to the configured Docker Hub namespace.
- The stored secret is valid.
- The image names in `docker-compose.yml` match that namespace.

### Reset the MongoDB data

If you intentionally want a completely clean database:

```bash
docker compose down -v
docker compose up -d --build
```

This deletes all existing Todo records.

## Security Notes

- Prefer a Docker Hub access token instead of an account password.
- Store secrets in Jenkins Credentials, never in `Jenkinsfile` or source control.
- Restrict access to Jenkins and the Docker daemon.
- Do not expose MongoDB port `27017` publicly in a production deployment.
- Disable Flask debug mode before production use.
- Pin container image and dependency versions for reproducible builds.
- Add automated tests and security scanning to the pipeline.
- Use TLS and a production frontend/backend server before internet-facing deployment.

## Possible Improvements

- Add React and Flask automated tests to Jenkins.
- Add dependency and container-image security scanning.
- Add pipeline quality gates.
- Replace mutable `latest` tags with versioned image tags.
- Add Jenkins webhook-based automatic triggering.
- Deploy the frontend as a static production build behind Nginx.
- Run Flask behind a production WSGI server.
- Remove public MongoDB port exposure when external access is unnecessary.
- Add monitoring, centralized logs, and rollback support.

## Author

**Amine Rahbani**
