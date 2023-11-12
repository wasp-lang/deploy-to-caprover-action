# Deploy Wasp Apps to CapRover

Github Action to deploy Wasp Apps to CapRover


## Usage

### Setting up CapRover

You'll need a [Caprover](https://caprover.com/) server setup.

Then create three apps for your Wasp app:

1. `server`
2. `client`
3. Postgres DB
    - go to one-click apps/dbs and create a vanilla PostgreSQL

Set the container ports:
- `server` app -> `3001`
- `client` app -> `8043`

Enable `HTTPS` and force redirects to HTTPS.

The env variables you will need to set:

- `server` app
  - `DATABASE_URL` (the DB you created on Caprover)
      - `postgres://postgres:<pw>@srv-captain--<db-app-name>:5432/postgres`
  - `WASP_WEB_CLIENT_URL` the client url (custom domain or the Caprover assigned subdomain)
  - `JWT_SECRET`
- `client` app
  - `SERVER_APP_URL` the server url

#### Cloudflare Notes

Set the SSL/TLS to `Full (strict)`.

Change the `A` recrod to `DNS only` when you try to verify the domain (Clicking `Enable HTTPS` tries to verify the domain and it fails if the `A` record is proxied).

When you get a certificate for the domain, switch the `A` recrod back to proxied.

### Setting up the workflow

Add `.github/workflows/caprover.yml` to your repository:

```yaml
name: Deploy to Caprover

on:
  push:
    branches:
      - "master"

jobs:
  build-and-deploy:

    permissions:
      contents: read
      packages: write

    uses: wasp-lang/deploy-to-caprover-action/.github/workflows/deploy.yml@main
    secrets:
      CAPROVER_SERVER: ${{ secrets.CAPROVER_SERVER }}
      SERVER_APP_NAME: ${{ secrets.SERVER_APP_NAME }}
      SERVER_APP_TOKEN: ${{ secrets.SERVER_APP_TOKEN }}
      CLIENT_APP_NAME: ${{ secrets.CLIENT_APP_NAME }}
      CLIENT_APP_TOKEN: ${{ secrets.CLIENT_APP_TOKEN }}
      SERVER_APP_URL: ${{ secrets.SERVER_APP_URL }}

      DOCKER_REGISTRY: ghcr.io
      DOCKER_REGISTRY_USERNAME: ${{ github.actor }}
      DOCKER_REGISTRY_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
```

Make sure to set the following secrets in your repository settings:

- `CAPROVER_SERVER`: The URL of your CapRover server

- `SERVER_APP_NAME`: The name of your server app
- `SERVER_APP_TOKEN`: The token of your server app
- `SERVER_APP_URL`: The URL of your server app

- `CLIENT_APP_NAME`: The name of your client app
- `CLIENT_APP_TOKEN`: The token of your client app

- `DOCKER_REGISTRY`: The URL of your docker registry (`ghcr.io` for GitHub Container Registry)
- `DOCKER_REGISTRY_USERNAME`: The username of your docker registry (`${{ github.actor }}` for GitHub Container Registry)
- `DOCKER_REGISTRY_PASSWORD`: The password of your docker registry (`${{ secrets.GITHUB_TOKEN }}` for GitHub Container Registry)