# A Little More Light

A small, self-contained motivational website with an illustrated twilight
landscape, gentle aurora, and an interactive collection of original quotes.
Everything is included locally; the page does not rely on external fonts,
images, or scripts.

## Preview locally but i have to push it online as well

```sh
docker build -t a-little-more-light:local .
docker run --rm -p 3001:80 a-little-more-light:local
```

Open `http://localhost:3001/`. The deployed `mot` container uses port `3000`.

`VERSION` (`1.4`) is the initial version baseline. Each successful push to
`main` increments the minor version above the latest numeric Git tag and
publishes only that numeric image tag. A matching Git tag records each
published version. Compose uses
`IMAGE_TAG` from `.env` and replaces the container named `mot`. For a local
pull and update, create `.env` in this directory:

```dotenv
DOCKERHUB_USERNAME=ishaq0925
IMAGE_NAME=lastpassword
IMAGE_TAG=1.5
```

Then run:

```sh
docker compose pull
docker compose up -d
docker compose ps
```

The website is available at `http://localhost:3000/`.

## Push-to-deploy workflow

Every push to `main` validates and builds the website image, publishes the next
numeric version tag to Docker Hub, then pulls that tag back onto the local
Docker machine and replaces its `mot` container. The deployment waits for the
container health check before succeeding.

### Required GitHub setup

1. Add a Docker Hub personal access token with **Read & Write** permission as
   the `DOCKERHUB_TOKEN` repository secret in **Settings → Secrets and
   variables → Actions**. Never commit the token or use an account password.
2. Register the Windows machine that runs Docker as a **self-hosted GitHub
   Actions runner** for this repository. Leave the standard `self-hosted`,
   `windows`, and `x64` labels enabled.
3. Keep Docker Desktop running and ensure the runner account can use Docker
   and Docker Compose. The deployment job runs on this machine and needs
   outbound access to GitHub and Docker Hub.
4. Ensure port `3000` is available. The deployment replaces the existing
   `mot` container and serves the site at `http://localhost:3000/`.

The workflow runs only on pushes to `main`; pull requests do not execute on
the self-hosted machine. Since the runner can control Docker on your machine,
restrict write access to this repository to trusted collaborators.

## Image version sequence

The workflow increments the minor number on every successful push to `main`.
`VERSION` is the starting baseline; after each publish, a matching numeric Git
tag is recorded so the next run continues the sequence.
