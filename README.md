# A Little More Light

A small motivational website served with Nginx. The page includes a twilight
illustration, an aurora effect, and a collection of original quotes. It is
self-contained and does not use external fonts, images, or JavaScript libraries.

## Run it locally

From this folder, build the image:

```powershell
docker build -t a-little-more-light:local .
```

Then start a temporary container:

```powershell
docker run --rm -p 3001:80 a-little-more-light:local
```

Open `http://localhost:3001` in a browser.

## Run with Docker Compose

Create a `.env` file in this folder:

```dotenv
DOCKERHUB_USERNAME=ishaq0925
IMAGE_NAME=lastpassword
IMAGE_TAG=1.12
```

Pull the selected image and start the website:

```powershell
docker compose pull
docker compose up -d
docker compose ps
```

The Compose deployment names the container `mot` and publishes the website on
`http://localhost:3000`.

The image tag must be provided with `IMAGE_TAG`. Docker Compose does not choose
it automatically.

## Automatic deployment

The GitHub Actions workflow in `.github/workflows/deploy.yml` runs after every
push to the `main` branch.

It:

1. Validates `index.html`.
2. Finds the next numeric image version.
3. Builds and publishes the image to Docker Hub.
4. Creates a matching Git tag.
5. Pulls the new image on a Windows self-hosted runner.
6. Replaces the `mot` container and waits for its health check.

The current Docker Hub image is:

```text
ishaq0925/lastpassword
```

## Versioning

The `VERSION` file is the starting version baseline (`1.8`). The latest
published tag is `1.11`, so the next image version is `1.12`.

The workflow increments the minor version for each successful push to `main`
and creates an annotated Git tag for each published image:

```text
1.11 -> 1.12 -> 1.13 -> ...
```

## GitHub Actions setup

### Docker Hub

Create a Docker Hub personal access token with Read & Write permission. Add it
to the GitHub repository as the `DOCKERHUB_TOKEN` secret. Do not put the token
in the workflow or source code.

### Windows runner

Register a Windows machine as a self-hosted GitHub Actions runner for this
repository. The runner must have:

- Docker Desktop running
- Docker Compose available
- Access to GitHub and Docker Hub
- Permission to control Docker
- Port `3000` available

The workflow uses the standard `self-hosted`, `windows`, and `x64` labels.
Because the runner can manage Docker on your machine, limit repository write
access to trusted collaborators.

## Deployment notifications

The workflow can email the publish and deployment results. Configure these
repository secrets:

| Secret | Description |
| --- | --- |
| `DEPLOYMENT_NOTIFICATION_EMAIL` | Address that receives the notification |
| `MAIL_SERVER` | SMTP server hostname |
| `MAIL_PORT` | SMTP port, usually `587` |
| `MAIL_USERNAME` | SMTP account or sender address |
| `MAIL_PASSWORD` | SMTP password or app password |

For Gmail, use an App Password instead of your regular account password. If
the mail secrets are not configured, the workflow warns and skips the email.
SMTP authentication or delivery errors are reported by the notification job.

## Useful commands

View the running container:

```powershell
docker compose ps
```

View website logs:

```powershell
docker compose logs --no-color website
```

Restart the website:

```powershell
docker compose restart website
```

Stop and remove the container:

```powershell
docker compose down
```
