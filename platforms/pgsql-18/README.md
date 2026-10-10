<div id="top-header" style="with:100%;height:auto;text-align:right;"></div>

# POSTGRE 18

- [./back](../../README.md)
- [Features](#features)
- [Configuration](#configuration)
- [Management](#management)
<br>

## <a id="features"></a>Features

![Alpine Linux](https://shields.io/badge/Alpine_Linux-%230D597F.svg?style=for-the-badge&logo=alpine-linux&logoColor=white)
![PostgreSQL](https://shields.io/badge/postgresql-4169e1?style=for-the-badge&logo=postgresql&logoColor=white)

Content:
- Linux Alpine 3.24
- PostgreSQL 18
<br><br>

Sources:
- https://github.com/docker-library/postgres
- https://github.com/docker-library/postgres/tree/master/18/alpine3.24

### Maintained by: [the PostgreSQL Docker Community](https://github.com/docker-library/postgres)

This is the Git repo of the [Docker "Official Image"](https://github.com/docker-library/official-images#what-are-official-images) for [postgres](https://hub.docker.com/_/postgres/) (not to be confused with any official `postgres` image provided by `postgres` upstream). See [the Docker Hub page](https://hub.docker.com/_/postgres/) for the full readme on how to use this Docker image and for information regarding contributing and issues.

The [full image description on Docker Hub](https://hub.docker.com/_/postgres/) is generated/maintained over in [the docker-library/docs repository](https://github.com/docker-library/docs), specifically in [the postgres directory](https://github.com/docker-library/docs/tree/master/postgres).

### See a change merged here that doesn't show up on Docker Hub yet?

For more information about the full official images change lifecycle, see [the "An image's source changed in Git, now what?" FAQ entry](https://github.com/docker-library/faq#an-images-source-changed-in-git-now-what).

For outstanding postgres image PRs, check [PRs with the "library/postgres" label on the official-images repository](https://github.com/docker-library/official-images/labels/library%2Fpostgres). For the current "source of truth" for [postgres](https://hub.docker.com/_/postgres/), see [the library/postgres file in the official-images repository](https://github.com/docker-library/official-images/blob/master/library/postgres).
<br><br>

## <a id="configuration"></a>Service Configuration

There is a dedicated GNU Make file and the main Docker directory with the required scripts to build the required platform configuration.

This service platform is designed to be managed by a parent `./../../.env` but, as it feeds `./docker/.env` through GNU Make, it can be edited manually.

If no GNU Make is in used, create a copy from the example `./docker/.env.example` into `./docker/.env`.

The service container can be easily manage by the `./docker/docker-compose.yml`.

Require environment variables at `./docker/.env` *(all are customizable)*:
```bash
COMPOSE_PROJECT_LEAD=myproj                             # <- lead abbreviation or acronym as part of related containers naming rule -------------------------> #
COMPOSE_PROJECT_CNET=mp-dev                             # <- useful for networking to connect between containers --------------------------------------------> #
COMPOSE_PROJECT_IMGK=alpine-3.24-pgsql-18               # <- real main image keys to manage automations for sharing resources -------------------------------> #
COMPOSE_PROJECT_NAME=mp-pgsql-dev                       # <- container name to build the service - it is important to set the environment in this variable --> #
COMPOSE_PROJECT_HOST=127.0.0.1                          # <- machine hostname referrer - not necessary for this project -------------------------------------> #
COMPOSE_PROJECT_PORT=7710                               # <- local machine port opened for container service ------------------------------------------------> #
COMPOSE_PROJECT_CPUS=2.00                               # <- container's maximum CPUs usage to apply by docker-compose - leave it empty for full usage ------> #
COMPOSE_PROJECT_MEMO=128M                               # <- container's maximum RAM usage to apply by docker-compose ---------------------------------------> #
COMPOSE_PROJECT_SWAP=256M                               # <- container's RAM swap space in storage executed by automation command ---------------------------> #
COMPOSE_PROJECT_DATA="./data"                           # <- platform data storage in local -----------------------------------------------------------------> #
COMPOSE_PROJECT_LOGS="./logs"                           # <- platform logs storage in local -----------------------------------------------------------------> #
POSTGRES_ROOT_PASSWORD="root-strong-password"           # <- database root password -------------------------------------------------------------------------> #
POSTGRES_DATABASE=local_dev                             # <- database name ----------------------------------------------------------------------------------> #
POSTGRES_USER=user_dev                                  # <- database user ----------------------------------------------------------------------------------> #
POSTGRES_PASSWORD="password-dev"                        # <- database password ------------------------------------------------------------------------------> #
```
<br>

### Containers Access Modes

- Set the required environment values in `./docker/.env` from `./docker/.env.example` if no GNU Make will be applied.
- Set the required configuration files by coping and updating them depending on your project requirements.
- Container availability by building the container with `docker-composer.yml` in separated configuration layers
    - Stand-alone
        - The container is intended to be published directly and accessed from the host network, typically via `0.0.0.0:<port>`. It does not require a shared Docker network. It is a common setting for local development.
    - Inside a Custom Network
        - The container is attached to a custom Docker network and is intended to be accessed through a reverse proxy or other containers on the same network.
        - This network setting is useful for isolating services while still allowing container-to-container communication.
        - It is a strongly recommended setting for remote deployment to avoid exposing the localhost port in used and protect by firewall.
        - <b>Connect from one container to another inside the custom network, by container name and its own exposed port</b>.
    - Host-Gateway
        - The container can reach services running on the host machine using the Docker host gateway mapping. This is useful when the container must access local services on the VPS/host, while public access is still handled through a reverse proxy. It is a recommended setting for remote deployment too.
    - Public exposure is controlled by the `ports` mapping.
    - `0.0.0.0:<port>` means externally accessible.
    - `127.0.0.1:<port>` means local-only access on the host and requires a reverse proxy, e.g. NGINX.
    - Docker network attachment controls container-to-container communication.
    - Host-gateway controls container-to-host communication.

build: ## builds and ensures changes in the Dockerfile, build steps, or copied-in files are applied. Not --no-recreate
	cd docker && $(DOCKER_COMPOSE) -f docker-compose.yml -f docker-compose.public.yml up --build -d

network: ## starts up into an existing custom network for container-to-container communication, runnning in detached mode
	cd docker && $(DOCKER_COMPOSE) -f docker-compose.yml -f docker-compose.network.yml up -d
<br><br>

## <a id="management"></a>Service Management

To manage the container, run the GNU Make recipes
```bash
$ make help
Usage: $ make [target]
Targets:
$ make help                           shows this Makefile help message
$ make port-check                     shows this project port availability on local machine
$ make env                            checks if docker .env file exists
$ make env-set                        sets docker .env file
$ make info                           shows container information
$ make ssh                            enters the container shell
$ make up                             starts and recreates containers if compose config or image changed, runnning in detached mode
$ make build                          builds and ensures changes in the Dockerfile, build steps, or copied-in files are applied. Not --no-recreate
$ make network                        starts up into an existing custom network for container-to-container communication, runnning in detached mode
$ make start                          starts the container and put on running from latest configuration
$ make stop                           stops the running container but data will not be destroyed
$ make restart                        restarts the running container
$ make clear                          removes container from Docker running containers
$ make destroy                        delete container image from Docker cache
$ make dev                            sets a development enviroment
$ make test-up                        creates a side database for tests
$ make test-down                      drops the side database for tests
$ make sql-install                    imports [file-name].sql file into database container as an initial project database located in /[dir-path]/
$ make sql-install-clear              clears up the [file-name].sql file as an initial project database located in /[dir-path]/
$ make sql-replace                    replaces the database container with the located in /[dir-path]/[file-name].sql file
$ make sql-backup                     exports into [file-name].sql file the project database backup into /[dir-path]/
$ make sql-backup-clear               clears up the project database backup [file-name].sql file located in /[dir-path]/
$ make sql-drop                       drops the current database and creates an empty new one inside the container
$ make remote-host-copy               copies referenced remote database by tcp connection into /[dir-path]/[file-name].sql for local development
$ make remote-file-copy               copies referenced remote database file into /[dir-path]/[file-name].sql for local development
```
<br>

See database version
```bash
$ make db-ssh

> cat /etc/os-release # see linux alpine version

> psql --version # see postgre version
```
<br><br>


## Contributing

Contributions are very welcome! Please open issues or submit PRs for improvements, new features, or bug fixes.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/YourFeature`)
3. Commit your changes (`git commit -am 'feat: Add new feature'`)
4. Push to the branch (`git push origin feature/YourFeature`)
5. Create a new Pull Request
<br><br>

## License

This project is open-sourced under the [MIT license](LICENSE).

<!-- FOOTER -->
<br>

---

<br>

- [GO TOP ⮙](#top-header)

<div style="with:100%;height:auto;text-align:right;">
    <img src="./../../resources/docs/images/pr-banner-long.png">
</div>