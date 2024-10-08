# mindsdb-demo-studio

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
3. Navigate to http://localhost:47344


# How to uninstall/clean up the MindsDB Docker Desktop Extension

This demo provides its own copy of the MindsDB Studio container, integrated into the Docker bridge network `demo-network` that all the other data services also use.

If you installed the MindsDB Docker Desktop Extension, there are some additional steps to remove the artifacts Docker Desktop installs as long-running services for you.

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