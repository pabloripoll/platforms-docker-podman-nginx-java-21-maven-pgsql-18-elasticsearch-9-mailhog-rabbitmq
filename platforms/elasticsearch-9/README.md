<div id="top-header" style="with:100%;height:auto;text-align:right;">
    <img src="./../../resources/docs/images/pr-banner-long.png">
</div>

# ELASTICSEARCH 9+

- [./main](../../README.md)
- [Features](#features)
- [Configuration](#configuration)
- [Management](#management)
<br>

## <a id="features"></a>Features

![Redhat](https://shields.io/badge/Red_Hat-EE0000?style=for-the-badge&logo=redhat&logoColor=white)
![Elasticsearch](https://img.shields.io/badge/-ElasticSearch-005571?style=for-the-badge&logo=elasticsearch)

Elasticsearch is a popular, distributed, open-source search and analytics engine built on Apache Lucene. Designed for horizontal scalability, reliability, and real-time management, it allows developers to store, search, and analyze massive volumes of data quickly and in near real-time. It is commonly used to power complex search features, log analytics, and real-time business intelligence metrics.
Content:

* Linux RHEL 9+
* Elasticsearch 9+
<br><br>

Sources:
* [elastic.co](https://www.elastic.co/)
* https://github.com/elastic
* https://github.com/elastic/dockerfiles
* https://hub.docker.com/hardened-images/catalog/dhi/elasticsearch
<br><br>

## Key Differences from Traditional Databases

To understand Elasticsearch, it helps to see how it differs from traditional relational databases (like MySQL or PostgreSQL):
```
# Feature           # Relational (e.g., MySQL)                          # Elasticsearch (Search Engine/NoSQL)
Data Structure      Tables with strict rows and columns                 Indices holding inverted indexes and JSON documents
Schema              Rigid: All rows must have the same columns          Dynamic/Structured: Mappings define fields, but variants are tolerated
Scalability         Usually scaled vertically                           Scaled horizontally natively across shards and nodes
Best For            Highly structured data with strict relationships    Full-text search, complex filter analytics, and log tracking
```

## Core Components of Elasticsearch

* Cluster: A collection of one or more connected server instances (nodes) that together hold your entire data inventory.
* Index: The equivalent of a "table" in a SQL database. It is a logical namespace used to group and partition related documents.
* Documents: The fundamental unit of data in Elasticsearch, stored as flat JSON objects. They consist of key-value fields. For example:
    ```
    {
        "_id": "12345",
        "title": "Alice in Wonderland",
        "author": "Lewis Carroll",
        "publish_year": 1865,
        "tags": ["fiction", "fantasy"]
    }
    ```

## Why Developers Use It

* Near Real-Time Search: It provides sub-second latencies for finding text phrases, filtering attributes, or computing auto-complete terms across billions of entries.
* Inverted Indexing: It maps individual tokens and words directly to their root source documents, enabling extremely powerful full-text and fuzzy search features out of the box.
* Advanced Aggregations: Developers can run complex analytics, calculate moving averages, and extract deep statistical trends on the fly without heavy performance penalties.
<br><br>

## Elasticsearch Client

### Kibana

Because Kibana is a heavy interface, running it outside of this specific container setup gives you a few distinct paths:

* **Own Container**: You can spin up a standalone Kibana container and link it to the custom network (PROJECT_CNET) defined in this repository's .env file. You simply point Kibana's environment variable to the container name and port of your Elasticsearch instance (e.g., http://mp-elastic-dev:[selected-port]).

* **Local App (Standalone)**: You can download the native Kibana archive for your OS from the [Official Elastic Download Page](https://www.elastic.co/downloads/kibana). Once extracted, you configure its kibana.yml file to point to http://127.0.0.1:[selected-port].

### Elasticvue (Lightweight Alternative)

As the repository notes, [Elasticvue](https://elasticvue.com/) is an excellent, lightweight alternative to Kibana. It is often preferred for local development because it doesn't require running a massive secondary background service:

* **Browser Extension (Easiest)**: You can install Elasticvue directly as a Chrome, Firefox, or Edge extension. Once installed, you open the extension, type in http://localhost:[selected-port], enter the ELASTIC_PASSWORD defined in your .env file, and you are instantly connected.

* **Local Web App / Container**: You can spin up the official [Docker Elasticvue Image](https://hub.docker.com/r/cars10/elasticvue) or download and configure a local version directly from the [Elasticvue GitHub Repository](https://github.com/cars10/elasticvue), feeding it your local machine's exposed port.

### How it ties into this Repository

Because of this repository's strategy, any local app, browser plugin, or external container can talk to it seamlessly as long as you match the port and pass the `ELASTIC_PASSWORD` string configured during the make up or make dev setup.
<br><br>

## <a id="configuration"></a>Service Configuration

There is a `./Makefile` to manege Docker container/s from outside `./docker` directory. Inside `./docker` there are the required scripts to build the platform service.

This service platform is also developed to be managed by a parent `Makefile` by its own `./../../.env` that feeds this `./docker/.env` and others for the infrastructure platforms. Repository directories structure overview:
```sh
.
├── platforms                       # infrastructure platforms
│   │
│   ├── elasticsearch-9             # this platform
│   │   ├── docker
│   │   │   ├── .env                # this platform environment
│   │   │   ├── docker-compose.yml
│   │   │   └── ...etc
│   .   └── Makefile                # this platform automation
│
├── .env                            # infrastructure platforms environments
├── Makefile                        # infrastructure platforms automations
.
```

If no GNU Make is in used, create a copy from the example `./docker/.env.example` into `./docker/.env`.

Required environment variables from `./docker/.env.example` for Docker Compose:
```bash
COMPOSE_LEAD=myproj                         # <- lead abbreviation or acronym as part of related containers naming rule -------------------------> #
COMPOSE_CNET=mp-dev                         # <- useful for networking to connect between containers --------------------------------------------> #
COMPOSE_IMGK=elastic-9                      # <- main container property for pulling and caching image into docker ------------------------------> #
COMPOSE_NAME=mp-elastic-dev                 # <- container name to build the service - it is important to set the environment in this variable --> #
COMPOSE_DATA="./elasticsearch-data"         # <- container persistant data on local machine -----------------------------------------------------> #
COMPOSE_LOGS="./logs"                       # <- container logs binding in local machine --------------------------------------------------------> #
COMPOSE_HOST=127.0.0.1                      # <- machine hostname referrer - not necessary for this project -------------------------------------> #
COMPOSE_CPUS=2.00                           # <- container's maximum CPUs usage to apply by docker-compose - leave it empty for full usage ------> #
COMPOSE_MEMO=1G                             # <- container's maximum RAM usage to apply by docker-compose ---------------------------------------> #
COMPOSE_SWAP=2G                             # <- container's RAM swap space in storage executed by automation command ---------------------------> #
COMPOSE_PORT=7712                           # <- local machine port opened for container service ------------------------------------------------> #
ELASTIC_SSL=false                           # <- elastic value to accept only ssl connections ---------------------------------------------------> #
ELASTIC_PASSWORD="password-dev"             # <- elastic password -------------------------------------------------------------------------------> #
COMPOSE_IGNORE_ORPHANS=true                 # <- stops docker compose from detecting orphaned containers ----------------------------------------> #
```

* **Ignore Orphans**:<br>
This Elasticsearch platform has been developed for optionally using it with a database client like Mongo Express. By avoiding the usage of `--remove-orphans` flag while building the container, it prevents to remove Elasticsearch container but terminal will show a warning message. To prevent that known message, the `.env` file has a constant value set to true `COMPOSE_IGNORE_ORPHANS=true` by default if using `./Makefile` for automation

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
<br><br>

## <a id="management"></a>Service Management

To manage the container, run the GNU Make recipes
```bash
$ make help
Usage: $ make [target]
Targets:
$ make help                           shows this Makefile help message
# -------------------------------------------------------------------------------------------------
#  Environment for Elasticsearch and Elasticsearch Client
# -------------------------------------------------------------------------------------------------
$ make ports-check                    shows this project port availability on local machine
$ make env                            checks if docker .env file exists
$ make env-set                        sets docker .env file
# -------------------------------------------------------------------------------------------------
#  Elasticsearch Container
# -------------------------------------------------------------------------------------------------
$ make info                           shows container information
$ make ssh                            enters the container shell
$ make up                             starts and recreates containers if compose config or image changed, runnning in detached mode
$ make build                          builds and ensures changes in the Dockerfile, build steps, or copied-in files are applied. Not --no-recreate
$ make network                        starts up into an existing custom network for container-to-container communication, runnning in detached mode
$ make start                          starts the container and put on running
$ make stop                           stops the running container but data will not be destroyed
$ make restart                        restarts the running container
$ make clear                          removes container from Docker running containers
$ make destroy                        delete container image from Docker cache
$ make dev                            sets a development enviroment
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