# mindsdb-demo-studio
Author: Michael Tanenbaum 
License: Apache 2.0

This repository provides a Docker Compose file for launching a MindsDB demo environment that includes MindsDB, RedPanda, Cassandra, and Postgres.

The intention of this demo platform is to provide a **quick** and **reliable** way to launch a MindsDB demo environment that includes a variety of data sources and sinks. 

MindsDB Demo Studio will include end-to-end examples that include front-end applications designed to illustrate how projects using MindsDB can be integrated into customer operational environments and workflows. For example, the MindsDB Demo Studio will include a simple web application that demonstrates how to use MindsDB to deliver RAG capabilities to internal teams. 

These demonstrations are intended to be delivered by technical sales/field engineering teams to prospects and customers. This is not intended for production use.

# Docker Compose File

This Docker Compose file includes the following services:
- **MindsDB**
- **RedPanda** (single broker deployment)
- **Cassandra** (single node deployment)
- **Postgres**

## Connectivity

Each service has connectivity to each other over named networks.

## Ports

Where possible, default ports have been preserved.

## Persistence

Because this is designed to be a quick way to launch demos for MindsDB, each service (including MindsDB) does not persist **anything** between container restarts.

## Setup and Launch Instructions


1. Clone this repo and navigate to the repo directory
2. Run: `docker-compose up -d`
3. Navigate to http://localhost:47334


# How to uninstall/clean up the MindsDB Docker Desktop Extension

This demo provides its own copy of the MindsDB Studio container, integrated into the Docker bridge network `demo-network` that all the other data services in this demo also use.

If you installed the MindsDB Docker Desktop Extension, you will have port conflict with the MindsDB container provided here **even if Docker Desktop and `docker ps -a` show no running containers**! Uninstalling the extension from the Docker Desktop leaves a Docker Nework service running that includes a MindsDB container bound to port 47334. 

Below are the steps for removing these artifacts. 

## Steps
1. Uninstall the MindsDB Docker Desktop extension from Docker Desktop UI.
2. Run `docker network ls` - you should see output similar to the below.
```
➜  mindsdb-demo-studio git:(main) ✗ docker network ls
NETWORK ID     NAME                                                         DRIVER    SCOPE
f20993b36fb1   bridge                                                       bridge    local
a8de80732472   host                                                         host      local
10253c21557c   mindsdb_mindsdb-docker-extension-desktop-extension_default   bridge    local
8c703939d9fd   none                                                         null      local
```
3. Next, run `docker network inspect <NETWORK ID>` with the network ID of the `mindsdb_mindsdb-docker-extension-desktop-extension_default`
4. Find the ID of the container in the "Containers" section of the Docker network JSON specification for the "mindsdb_service" (in this case, the long string starting `017e` below.)
```
        "Containers": {
            "017e000ca05b61efb19a3e5c4fab59561b5506783091f6ee0ef610d24ba70d2e": {
                "Name": "mindsdb_service",
                "EndpointID": "527a0112704aff45842b56fc83bae430ce3fce6b125eddfc73553a9e77fbc15f",
                "MacAddress": "02:42:ac:15:00:02",
                "IPv4Address": "172.21.0.2/16",
                "IPv6Address": ""
            }
        },
```
5. Run `docker stop <Container ID>`
6. Run `docker rm <Container ID>`
7. Run `docker network rm <Network ID>`
8. Confirm network was removed. Run: `docker network ls`.