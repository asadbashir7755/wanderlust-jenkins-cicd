# Wanderlust Jenkins CI/CD Pipeline

A Jenkins pipeline that tests, builds, publishes and deploys a containerised MERN
application. The pipeline is the project here. The application is just what it
deploys.

The Wanderlust app itself is from
[krishnaacharyaa/wanderlust](https://github.com/krishnaacharyaa/wanderlust). The
pipeline, container setup and deploy scripts are mine.

Portfolio: [committodeploy.dev](https://committodeploy.dev)

## Why I built it

Running a pipeline on a self-hosted Jenkins agent means dealing with things a
hosted runner hides from you: agent setup, Docker permissions, credential
injection, registry auth, health checks and rollback. I wanted all of that to be
mine to get right.

## The pipeline

Declarative pipeline in [`jenkinsfile.ci`](jenkinsfile.ci), running on a
self-hosted agent labelled `ubuntuagent`.

| Stage | What it does |
|---|---|
| Checkout | Clones the branch being built |
| Backend test | `npm ci`, then unit and integration tests |
| Frontend test | Frontend test suite |
| Build | Builds backend and frontend images in parallel |
| Push | Logs into Docker Hub and pushes the tagged images |
| Deploy | Writes `.env` from Jenkins credentials, brings up the stack with Compose |

### Secrets

Nothing is committed. Every secret comes from the Jenkins credential store at run
time through `withCredentials`:

```
JWT_SECRET
MONGO_INITDB_ROOT_USERNAME
MONGO_INITDB_ROOT_PASSWORD
GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET
REDIS_PASSWORD
dockerhublogin
```

[`.env.example`](.env.example) lists every variable the stack needs. Copy it to
`.env` for local runs. `.env` is gitignored.

### Shared library

The reusable pipeline steps live in a separate repo,
[jenkins-shared-library](https://github.com/asadbashir7755/jenkins-shared-library):
`clone`, `backend_test`, `docker_build`, `docker_push`, `envconfig`.

## Infrastructure

```
infra/
  dockercompose.yml        frontend, backend, MongoDB, Redis
  dockercompose.prod.yml   production overlay with healthchecks
  nginx/                   reverse proxy config
scripts/
  wait-for-health.sh       waits until containers report healthy
  smoke-tests.sh           checks the running stack after deploy
  rollback.sh              redeploys the previous image tag
  postInstall.mjs
```

The deploy does not report success until `wait-for-health.sh` confirms the
containers are up and `smoke-tests.sh` passes against the live stack. If either
fails, `rollback.sh` puts the previous image back.

## Running it locally

```bash
git clone git@github.com:asadbashir7755/wanderlust-jenkins-cicd.git
cd wanderlust-jenkins-cicd

cp .env.example .env
# fill in real values, then:
docker compose -f infra/dockercompose.yml up -d --build

./scripts/wait-for-health.sh
./scripts/smoke-tests.sh
```

Frontend runs on port 3000, backend on port 8000.

## Setting up Jenkins

1. A Jenkins controller with an agent labelled `ubuntuagent` that has Docker and
   Node installed.
2. Add the credential IDs listed above to the Jenkins credential store.
3. Register the shared library as a Global Pipeline Library named
   `jenkins-shared-library`.
4. Point a Pipeline job at `jenkinsfile.ci`.

## Tech stack

Jenkins, Docker, Docker Compose, Nginx, Node.js, MongoDB, Redis, Bash, Docker Hub
