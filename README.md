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

The workflow runs `npm test` and builds the image on pull requests without
publishing. Pushes to `main` and manual runs publish
`ishaq0925/lastpassword:<commit-sha>` and `ishaq0925/lastpassword:latest` to
Docker Hub. It does not deploy to a VM.

Add a Docker Hub access token as the `DOCKERHUB_TOKEN` secret in this
repository's **Settings → Secrets and variables → Actions**. The workflow
uses the token only for non-pull-request builds. Never commit the token or
use your account password.
