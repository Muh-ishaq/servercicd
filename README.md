# Server API

A minimal Node.js HTTP API used to demonstrate a separate server repository and
Docker image build.

## Run locally

Requires Node.js 22 or newer.

```sh
npm test
npm start
```

The API listens on port `3000` (override with `PORT`). Endpoints:

- `GET /` returns a plain-text running message.
- `GET /health` returns `{"status":"ok"}`.
- Other paths return a JSON `404` response.

## Build and run the container

```sh
docker build -t server:local .
docker run --rm -p 3000:3000 server:local
```

The included Compose file uses a public Docker Hub image. For a local pull or
deployment, create `.env` in this directory:

```dotenv
DOCKERHUB_USERNAME=ishaq0925
IMAGE_TAG=latest
```

Then run:

```sh
docker compose pull
docker compose up -d
docker compose ps
```

The service is available at `http://localhost:3000/health`.

## GitHub Actions CI

The workflow runs on pull requests, pushes to `main`, and manual runs. It runs
`npm test`, builds the Docker image, and loads it into the runner's local
Docker engine. It does not log in to Docker Hub, upload an image, or deploy to
a VM. The GitHub-hosted runner is temporary, so the local image is available
only during that workflow job.
