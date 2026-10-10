<div id="top-header" style="with:100%;height:auto;text-align:right;">
    <img src="./../../resources/docs/images/pr-banner-long.png">
</div>

# NGINX + JAVA 21 + MAVEN

- [./main](../../README.md)
- [Features](#features)
- [Configuration](#configuration)
- [Management](#management)
<br>

## <a id="features"></a>Features

![Alpine Linux](https://shields.io/badge/Alpine_Linux-%230D597F.svg?style=for-the-badge&logo=alpine-linux&logoColor=white)
![Debian](https://shields.io/badge/debian-red?style=for-the-badge&logo=debian&logoColor=orange&color=darkred)
![Oracle](https://custom-icon-badges.demolab.com/badge/Oracle-F80000?style=for-the-badge&logo=oracle&logoColor=fff)
![Nginx](https://shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![Java](https://shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Maven](https://shields.io/badge/MAVEN-000000?style=for-the-badge&logo=apachemaven&logoColor=blue)

Content:
- Linux Alpine 3.24
- Linux Debian 12 Slim *(Optional Dockerfile)*
- [Linux Oracle 9](https://hub.docker.com/_/oraclelinux) *(Optional Dockerfile)*
- Java 21
- Maven
<br><br>

It can be installed the most known **JAVA** frameworks:

![Spring](https://shields.io/badge/Spring-6DB33F?style=for-the-badge&logo=spring&logoColor=white)
![Spring Boot](https://shields.io/badge/springboot-000000?style=for-the-badge&logo=springboot&logoColor=green)

- [Spring](https://spring.io/)
- [Spring Boot 3](https://spring.io/blog/2022/05/24/preparing-for-spring-boot-3-0)
- Others...
<br>

Take into account that each framework will demand its specific configuration from inside container.
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
COMPOSE_PROJECT_IMGK=alpine-3.24-nginx-java-21          # <- real main image keys to manage automations for sharing resources -------------------------------> #
COMPOSE_PROJECT_NAME=mp-apirest-dev                     # <- container name to build the service - it is important to set the environment in this variable --> #
COMPOSE_PROJECT_HOST=127.0.0.1                          # <- machine hostname referrer - not necessary for this project -------------------------------------> #
COMPOSE_PROJECT_PORT=7501                               # <- local machine port opened for container service ------------------------------------------------> #
COMPOSE_PROJECT_PATH="../../../api-rest"                # <- path where application is binded from container to local ---------------------------------------> #
COMPOSE_PROJECT_CPUS=2.00                               # <- container's maximum CPUs usage to apply by docker-compose - leave it empty for full usage ------> #
COMPOSE_PROJECT_MEMO=128M                               # <- container's maximum CPUs usage to apply by docker-compose - leave it empty for full usage ------> #
COMPOSE_PROJECT_SWAP=256M                               # <- container's maximum RAM usage to apply by docker-compose ---------------------------------------> #
COMPOSE_PROJECT_USER=osuser                             # <- container's project directory user -------------------------------------------------------------> #
COMPOSE_PROJECT_GROUP=osgroup                           # <- container's project directory group ------------------------------------------------------------> #
```

<font color="orange"><b>IMPORTANT:</b></font>

Although the project aims to keep platform settings consistent across machines, some runtime requirements can differ between environments *(for example: memory limits or which supervisord services should be active)*.

> Note: Though this platform aim is for REST API services, it also can be used for monolith web applications

### Containers Access Modes

- If no application is on `./api-rest` directory *(or your custom binded directory name)* once container is up it wont provide a application and therefore NGINX will respond with an error. Copy an start-up example application or create a parking page.
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
<br><br>

### Dockerfile

You might be using this repository with different databases connection. Copy from sample and set the correct packages and modules required by commenting them out, and others not required is recommended to be commented. In this way, the container will be built only with the neccessary settings an in less time

- `./docker/Dockerfile.Alpine` -> `./docker/Dockerfile`

Optionally, the platform directory could come with a Dockerfile with Debian version in case of AI preference usage.

### NGINX

The default example is a server block for a REST API, but it can be use for webapps too

- `./docker/config/nginx/conf.d-sample/default.conf` -> `./docker/config/nginx/conf.d/default.conf`

### JAVA

To automatically run the JAVA application, create the Supervisord service that runs the application. You can choose the dev or production version, and set it according to your project requirements

- `./docker/config/supervisor/conf.d-sample/java.conf` -> `./docker/config/supervisor/conf.d/java.conf`

```bash
[program:java]
command=java -jar /var/www/target/app.jar
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
autorestart=false
startretries=0
```
<br>

To support environment-specific differences, there is a sample supervisor configuration directory:
`./platform/nginx-java-21/docker/config/supervisor/conf.d-sample`

Before building the container you must copy that directory to:
`./platform/nginx-java-21/docker/config/supervisor/conf.d`

Make sure the copied conf.d contains at least the service files needed to run Nginx and Java (for example, supervisor program entries for both).

```bash
$ cd ./platform/nginx-java-21/docker/config/supervisor
$ cp -vn conf.d-sample/nginx.conf conf.d-sample/java.conf conf.d/
'conf.d-sample/nginx.conf' -> 'conf.d/nginx.conf'
'conf.d-sample/java.conf' -> 'conf.d/java.conf'
```

This approach lets developers run additional worker processes locally without changing the shared platform settings. If you need to update Nginx, Java, or supervisord configurations on a running container, there are Makefile recipes in `./platform/nginx-java-21/Makefile` that can apply changes *(reload or update services)* without destroying and rebuilding the container. Check the Makefile for available targets and usage by executing `$ make help` in its directory.
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
$ make host-gateway                   starts up for container-to-host communication, runnning in detached mode
$ make start                          starts the container and put on running from latest configuration
$ make stop                           stops the running container but data will not be destroyed
$ make restart                        restarts the running container
$ make clear                          removes container from Docker running containers
$ make destroy                        delete container image from Docker cache
$ make dev                            sets a development enviroment
$ make supervisord-conf               lists supervisord services set on the running container
$ make supervisord-update             updates supervisord services without the need of stoping or rebuilding the container
$ make nginx-conf                     shows nginx configuration set on the running container
$ make nginx-update                   updates nginx configuration without the need of stoping or rebuilding the container
$ make nginx-default-conf             shows nginx default server block set on the running container
$ make nginx-default-update           updates nginx default server block without the need of stoping or rebuilding the container
```
<br><br>


---

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