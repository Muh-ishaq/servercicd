# Server API

A minimal Node.js HTTP API used to demonstrate a separate server repository and
automated Docker deployment.

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
DOCKERHUB_USERNAME=your-dockerhub-user-or-org
IMAGE_TAG=latest
```

Then run:

```sh
docker compose pull
docker compose up -d
docker compose ps
```

The service is available at `http://localhost:3000/health`.

## GitHub Actions CI/CD

`.github/workflows/deploy.yml` runs tests and builds the container on pull
requests targeting `main`; pull requests do not publish or deploy. A push to
`main` or a manual workflow run publishes two public Docker Hub tags:

- `DOCKERHUB_USERNAME/server:<commit-sha>`
- `DOCKERHUB_USERNAME/server:latest`

After publishing, the workflow runs on a private self-hosted GitHub Actions
runner on the server VM. It writes the commit SHA into `.env`, pulls that exact
image, recreates the Compose service, and sends a Slack notification if
configured.

### GitHub configuration

In this repository's **Settings → Secrets and variables → Actions**, add:

| Type | Name | Value |
| --- | --- | --- |
| Variable | `DOCKERHUB_USERNAME` | Docker Hub user or organization that owns the public `server` image |
| Secret | `DOCKERHUB_TOKEN` | Docker Hub access token with permission to push |
| Secret (optional) | `SLACK_WEBHOOK_URL` | Slack incoming webhook URL |

Create the public Docker Hub repository `server` under that user or
organization. Use an access token instead of a Docker Hub password.

### Server VM runner

Install Docker Engine and the Docker Compose v2 plugin on the Linux server VM.
In this repository, go to **Settings → Actions → Runners → New self-hosted
runner**, follow GitHub's registration instructions on the VM, add the custom
runner label `server-vm`, and install/start the runner as a service. The
workflow requires labels `self-hosted`, `linux`, and `server-vm`; confirm the
runner is online before deploying. The runner account must be able to run
`docker compose` non-interactively.

The VM must be able to access GitHub and Docker Hub over HTTPS. Open port `3000`
in the VM/network firewall only if the API should be reachable directly. Do
not run untrusted pull request code on this deployment runner.

## Roll back

To redeploy an older version, set `IMAGE_TAG` in the VM checkout's `.env` to a
previously published commit SHA and run `docker compose pull` followed by
`docker compose up -d`. The normal workflow always deploys the current commit's
immutable SHA tag.
