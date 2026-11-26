# OpenGeoPortal

An open-source, self-hosted portal for geographic data, maps and apps.
Content builds on itself: **Data → Services → Layers → Maps → Apps**.

This repository runs the portal with Docker Compose. The portal itself is built in
[opengeoportal-org/opengeoportal](https://github.com/opengeoportal-org/opengeoportal) and published as images on
the GitHub Container Registry:

| Part | Where | Built with |
|---|---|---|
| Running it | `opengeoportal-deploy` (this one) | Docker Compose, Keycloak realm, nginx and database settings |
| Web client | `ghcr.io/opengeoportal-org/client`, from `client/` | Angular, served by nginx |
| API | `ghcr.io/opengeoportal-org/api`, from `api/` | .NET 10 |
| Database | `postgis/postgis` | PostgreSQL 17 + PostGIS |
| Sign-in | `quay.io/keycloak/keycloak` | Keycloak 26 |

The images run on `linux/amd64` and `linux/arm64`.

## Getting started

You need [Docker](https://docs.docker.com/get-docker/) with Docker Compose, and this repository. Nothing else:
the images are downloaded.

1. **Get this repository**

   ```bash
   git clone https://github.com/opengeoportal-org/opengeoportal-deploy.git opengeoportal
   cd opengeoportal
   ```

2. **Create your settings file**

   Make a copy of `.env.example` and name the copy `.env`: copy and paste the file in the same folder, then rename
   the copy. Or with a command:

   ```bash
   cp .env.example .env
   ```

   On Windows (PowerShell or Command Prompt): `copy .env.example .env`

   Open `.env` and change the passwords. Is port 8080 already in use on your machine? Change
   `OGP_PORT` and the port in `OGP_PUBLIC_URL`.

3. **Start everything**

   ```bash
   docker compose up -d
   ```

   The first time this takes a few minutes: the images are downloaded, the database is created,
   and Keycloak prepares itself. The command returns when everything is ready.

4. **Open the portal** at `OGP_PUBLIC_URL` (default http://localhost:8080).
   The setup wizard opens: name your organisation and create your administrator account, then sign in.

   Whoever opens a new portal first gets this wizard. On a server that is reachable from the internet,
   finish the setup right after starting it.

## What is in here

| File | What |
|---|---|
| `docker-compose.yml` | Everything on one machine: database, Keycloak, API and web client |
| `docker-compose.external.yml` | Only the API and web client, for your own database and Keycloak |
| `.env.example` | Every setting, with explanations; copy it to `.env` |
| `docker/keycloak/ogp-realm.json` | The Keycloak realm (clients, roles, the API's service account), imported on the first start |
| `docker/nginx/keycloak.conf` | Serves Keycloak under `/auth` through the web client's nginx |
| `docker/postgres/initdb/` | Creates Keycloak's database next to the portal's on the first start |

## Useful addresses

| Address | What |
|---|---|
| `/` | The portal |
| `/health` | Status of the API, database and Keycloak |
| `/api/docs` | The API documentation: every endpoint, to read and try. Off with `OGP_API_DOCS=false` in `.env` |
| `/auth/admin` | Keycloak admin console (user `KEYCLOAK_ADMIN` from `.env`). People are managed in the portal (Settings > People); this is for Keycloak itself |

## Versions

`OGP_VERSION` in `.env` chooses the image version, the same for the API and the web client:

| Value | Runs |
|---|---|
| `latest` | The newest release (default) |
| `1` | The newest 1.x.y: new features and fixes, nothing that needs action from you |
| `1.0` | The newest 1.0.x: fixes only |
| `1.0.4` | Exactly that version |

On a server that should not change on its own, pin `1` or `1.0`.

This repository is tagged with the same versions (`v1.0.4`). To run exactly one version with the files made for it:
`git checkout v1.0.4` and `OGP_VERSION=1.0.4`.

## Updating

Download the new images and restart. Database changes are applied by the API when it starts:

```bash
git pull
docker compose pull
docker compose up -d
```

`git pull` brings changes to these files, such as a newer Keycloak realm; your `.env` stays as it is.


## Languages

The portal speaks English, Albanian, Macedonian, Dutch, German, French, Spanish, Portuguese, Italian, Turkish,
Arabic, Chinese and Japanese. Everyone picks their own in the top bar; an administrator sets the language for people
who have not chosen one under Settings > Language.

Keycloak's sign-in page follows the language too (it has no Albanian or Macedonian yet: those sign in in English).
The realm in `docker/keycloak` turns this on for new installs. With your own Keycloak, turn it on in the admin console:
realm `ogp` > Realm settings > Localization > Internationalization on, and add the languages.

## QGIS

The OpenGeoPortal plugin for QGIS (in the `qgis-plugin` folder of the opengeoportal repository) brings portal layers and maps into QGIS and sends QGIS layers to
the portal. People sign in with their portal account in the browser. For that, the portal's Keycloak client accepts
the address `http://127.0.0.1/*` (the plugin's local sign-in answer); the realm here has it. With your own Keycloak, add it
in the admin console: realm `ogp` > Clients > `ogp-client` > Valid redirect URIs.

## Coordinate systems

Uploaded data is stored in WGS 84, so every map, layer and app can use it. The portal remembers the coordinate
system it came in: downloads, and layers added through the QGIS plugin or the OGC API, come back in that one,
with the same coordinates.

Most coordinate systems convert by a published formula. Some (older datums such as NAD27) need a transformation
grid: the API and the database fetch the ones they need from `cdn.proj.org` when first used and keep them, so
both convert the same way. Without internet access set `OGP_PROJ_NETWORK=OFF`; those conversions are then a
few metres off. Your own database (`docker-compose.external.yml`): set `PROJ_NETWORK=ON` in the environment of
its PostgreSQL server.

## Your own database or Keycloak

Already have PostgreSQL/PostGIS or Keycloak on another server? Fill in the last section of `.env`
and start only the portal itself:

```bash
docker compose -f docker-compose.external.yml up -d
```

The README of `portalapi` describes every setting.

## Starting over

This removes the containers, the database, uploaded files and the images. It keeps `.env`:

```bash
docker compose down -v --rmi all
```
